# AI/07 · Background Ingestion & Parsing Automation

> Status: Draft v1 · Temporal workflows in `services/workers/src/ingestion/*` · "Background AI platform parsing automation"

## 1. Goal

Keep a **fresh, normalized, enriched food graph** for every active merchant — menus, prices, availability, dish semantics, nutrition and allergen estimates — without scraping marketplaces.

## 2. Sources and adapters

| Source | Adapter | Trigger | Freshness |
|---|---|---|---|
| POS (Square, Toast, Clover) | `PosMenuAdapter` (official APIs, merchant-authorized) | Webhooks + nightly sync | Near real-time (items 86'd) |
| Partner console upload (PDF, photo, URL) | `UploadMenuAdapter` | On upload | On change |
| Merchant website (permission recorded) | `WebMenuAdapter` (Playwright; robots.txt respected; optional Cloudflare Browser Run/Browserbase for scale) | Weekly + on-demand | Weekly |
| WhatsApp catalog / menu photo sent by merchant | `ChatMenuAdapter` | On message | On change |
| Places data (hours, location, categories) | `PlacesAdapter` (license-compliant) | Monthly | Monthly |
| Availability signals from liaison replies | `LiaisonSignalAdapter` | Every check | Live |

Marketplace consumer apps (Uber Eats, DoorDash, Grubhub, PedidosYa) are **not** sources ([research](../00-product/02-research-and-partner-landscape.md)).

## 3. Pipeline

```text
MenuIngestionWorkflow(merchantId, source)
  1. fetch         → raw artifact to object storage (hash for dedupe)
  2. extract       → Claude vision/text → ParsedMenu (structured output)
                     sections, items, descriptions, prices, sizes, modifiers, labels, notes
  3. validate      → price sanity, duplicate items, currency, missing prices, modifier trees
  4. normalize     → map items → canonical dishes (embedding kNN + LLM adjudication)
  5. enrich        → DishEnrichmentWorkflow per new/changed item (Batch API when bulk)
  6. diff          → compare with current version: added/removed/price-changed items
  7. review gate   → low confidence or high-risk changes → partner console review / ops queue
  8. publish       → new menu_version, embeddings, search index; emit menu.version.published
```

### ParsedMenu (structured output, abbreviated)

```ts
const ParsedMenu = z.object({
  currency: z.enum(["USD", "CAD"]),
  sections: z.array(z.object({
    name: z.string(),
    items: z.array(z.object({
      name: z.string(),
      description: z.string().nullable(),
      prices: z.array(z.object({ label: z.string().nullable(), amount: z.number() })), // sizes
      modifiers: z.array(z.object({ group: z.string(), required: z.boolean(),
        options: z.array(z.object({ name: z.string(), priceDelta: z.number() })) })),
      declaredLabels: z.array(z.string()),     // only what the menu states: "V", "GF", "spicy", "halal"
      declaredAllergens: z.array(z.string()),  // only if printed on the menu
      confidence: z.number(),
    })),
  })),
  notes: z.array(z.string()),                  // "18% gratuity for 6+", "ask about allergens"
});
```

## 4. Enrichment (per item)

| Field | Method | Confidence handling |
|---|---|---|
| Canonical dish | Embedding kNN over `canonical_dishes` + LLM adjudication; create new canonical dish via review if no match | < 0.7 → review |
| Ingredients (typical) | LLM from name/description + canonical dish typical ingredients | Marked *estimated* |
| Allergen risk | Rules (ingredient → allergen) + LLM, per allergen probability | Never shown as "free of"; blocks anaphylaxis-level seats unless merchant-declared |
| Nutrition estimate | Map to FoodData Central recipe components × portion heuristics | Range + confidence; shown as "≈" |
| Flavor/texture profile | LLM scoring 0–1 on fixed axes | — |
| Spice level | Declared > inferred | — |
| Thermal class & travel score | Rules by dish type (fried, soup, raw, frozen) + LLM | Used by Logistics |
| Cultural & occasion tags | Canonical dish knowledge + LLM | — |
| Traditional-medicine properties | From ingredient knowledge (TCM nature, Ayurveda) aggregated | Only for lens users |
| Shareability, kid-friendliness | LLM + rules | — |

Bulk enrichment runs via **Message Batches** (50 % cost, results keyed by `custom_id = itemId:version`).

## 5. Quality controls

- **Golden menus:** 50 hand-labeled menus (diverse cuisines, languages, formats: PDF, phone photo, chalkboard) as an eval set; target item-level F1 ≥ 0.95 for names/prices.
- **Price change guard:** > 25 % change on an item → review.
- **Merchant confirmation:** partner console shows "What we understood" with editable fields; merchant-declared allergens override estimates.
- **Drift:** liaison replies that frequently mark an item unavailable → mark "often unavailable"; repeated substitution → suggest menu update to merchant.
- **Prompt-injection hygiene:** menu text is data; extraction prompts state it; outputs are schema-bound.

## 6. Background agents beyond menus

| Agent | Job | Cadence |
|---|---|---|
| **Hours & status watcher** | Detect closures/holiday hours from merchant messages, POS, places data | Daily |
| **Merchant reliability scorer** | Answer time, acceptance, accuracy, lateness → `merchant_reliability` | Hourly rollup |
| **Occasion calendar builder** | Compute moving festivals, sunset times per metro for the year | Yearly + monthly check |
| **Memory consolidation** | Summarize episodes, decay preferences, detect rituals | Nightly (Batch) |
| **Dial prior trainer** | Update dial priors per context cluster | Nightly |
| **Outreach prospector** (Phase 2) | Rank prospective restaurants for onboarding by demand gaps (cuisines asked for but unavailable) | Weekly |

## 7. Costs & limits

- Track per merchant: tokens per ingestion, re-ingestion frequency.
- Skip re-extraction when artifact hash unchanged; diff-only enrichment.
- Browser automation budget (if used for websites) capped per day; human-in-the-loop for login-gated sites is **not** used (we don't log into third-party ordering systems).
