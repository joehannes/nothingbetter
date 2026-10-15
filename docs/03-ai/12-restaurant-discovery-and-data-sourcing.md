# AI/12 · Restaurant Discovery & Data Sourcing (without marketplaces)

> Status: Draft v1 · 2026-10-15 · Answers: "Can we build a restaurant DB (contacts/WhatsApp, offerings, prices, hours) from Google Maps or similar, and can an AI investigate nearby restaurants per client?" · Related: [ADR-0011](../01-architecture/adr/ADR-0011-marketplace-data-collection.md), [AI/07](07-background-ingestion-and-parsing.md)

## 1. Short answer

**Yes for the directory, partly for menus, and only with consent for WhatsApp.**

| Data | Best legitimate source | Storable? | Coverage (measured / documented) |
|---|---|---|---|
| Restaurant exists, location, category | **Overture Maps places** (open data) | Yes (CDLA-Permissive-2.0 / Apache-2.0 / CC0 by source, attribution) | Miami-Dade box: **12,553** food places; Santo Domingo box: **2,286** (measured below) |
| Phone number | Overture `phones`; Google Places (runtime) | Overture yes; Google no (place ID only) | Overture: **88 %** Miami-Dade, **85 %** Santo Domingo |
| Website / social | Overture `websites`, `socials` | Yes | Websites: **79 %** Miami-Dade, **34 %** Santo Domingo; socials: 60 % / 98 % |
| Opening hours, price level, rating, open/closed status | **Google Places API** at request time | No (refresh on demand) | Google fields exist for these (Enterprise SKU) |
| Menus & prices | Restaurant's own website/PDF read by AI; menu sent over WhatsApp; owner-authorized Google Business Profile food menus; POS | Yes, with provenance (permission confirmed at onboarding) | Depends on website menus — measure in pilot zone |
| WhatsApp number | Explicit `wa.me` links (rare: < 1 % in both cities); otherwise ask the restaurant | Yes, after opt-in | Phones are plentiful; WhatsApp status unknown until the restaurant replies |

Prices don't need to be perfect up front: every order is **confirmed with the kitchen (items + prices) before the customer pays** ([WF/02](../02-workflows/02-availability-confirmation-whatsapp.md)), so draft menus are good enough to propose food.

## 2. Measurements (2026-10-15)

### Overture Maps places (release 2026-09-23.1), categories restaurant, fast_food_restaurant, cafe, coffee_shop, bakery, food_truck_stand, bar_and_grill

| Area (bounding box) | Food places | With phone | With website | With social link | Confidence ≥ 0.77 | Explicit WhatsApp link |
|---|---|---|---|---|---|---|
| Miami-Dade (-80.87…-80.12, 25.13…25.98) | 12,553 | 11,079 (88 %) | 9,941 (79 %) | 7,586 (60 %) | 10,032 (80 %) | 5 (0.0 %) |
| Gran Santo Domingo (-70.10…-69.75, 18.40…18.60) | 2,286 | 1,944 (85 %) | 781 (34 %) | 2,230 (98 %) | 824 (36 %) | 16 (0.7 %) |

Method: DuckDB over Overture's public Parquet (Azure mirror), bounding-box filter, `basic_category` in the list above. Bounding boxes approximate the areas; low confidence often means stale or closed places.

### OpenStreetMap (Overpass, 2026-10-15) — Miami-Dade County, amenity restaurant/fast_food/cafe

| Total | Phone | Opening hours | Website | Menu URL | WhatsApp tag | Cuisine |
|---|---|---|---|---|---|---|
| 2,531 | 54 % | 44 % | 55 % | 4.9 % | 0 % | 80 % |

OSM had about one fifth of Overture's count here but better cuisine tags; its `opening_hours` and `cuisine` are useful complements. (Santo Domingo Overpass queries failed through our proxy; not measured.)

### Is that "50 % of PedidosYa / Uber Eats"?

Marketplace restaurant counts per area aren't published, so a precise ratio can't be computed. For **discovery and contact**, open data very likely covers more restaurants than any single marketplace lists (marketplaces only list signed-up restaurants). The real gap is **menus and prices** — the pilot measures what share of zone restaurants have a machine-readable menu on their own website (§6).

## 3. Source details & terms

| Source | What it gives | Cost | Terms that matter |
|---|---|---|---|
| **Overture Maps places** ([guide](https://docs.overturemaps.org/guides/places/)) | ~81M places globally; names, `basic_category`/`taxonomy`, websites, socials, emails, phones, brand, addresses, confidence; sources incl. Meta (~59M), Microsoft, Foursquare, AllThePlaces | Free (Parquet on S3/Azure) | CDLA Permissive 2.0 / Apache 2.0 / CC0 depending on source; keep attribution |
| **OpenStreetMap** | Cuisine, hours, phone, website, `contact:whatsapp` tag ([wiki](https://wiki.openstreetmap.org/wiki/Key:contact:whatsapp)) | Free | ODbL share-alike for derived databases — keep OSM-derived fields in a separate layer or comply; don't hammer public Overpass (self-host for production) |
| **Google Places API (New)** ([fields](https://developers.google.com/maps/documentation/places/web-service/data-fields)) | Phones, website, regular/current opening hours, price level/range, delivery/takeout flags, reviews, review summaries; **no menu-items field** (only `menuForChildren`) | Nearby Search Enterprise $35 / 1,000 (1,000 free/month); Place Details Enterprise $20 / 1,000 (1,000 free) ([pricing](https://developers.google.com/maps/billing-and-pricing/pricing)) | "You must not pre-fetch, cache, or store Places API content beyond the allowed exceptions"; place IDs may be stored indefinitely; results on a map must be on a Google Map; Google logo/attribution required ([policies](https://developers.google.com/maps/documentation/places/web-service/policies)) → **use at request time, per client** |
| **Gemini API grounding with Google Maps** | LLM answers grounded in Maps data | 5,000 requests/month free, then $14 / 1,000 ([pricing](https://ai.google.dev/gemini-api/docs/pricing)) | Google Maps content/attribution terms apply; an alternative to Places calls for the research agent |
| **Google Business Profile API** — `accounts.locations.getFoodMenus` | The restaurant's own Google menu (sections, items, prices) | Free (API access approval) | Requires the owner's OAuth consent (`business.manage`) → "Connect your Google profile" at onboarding |
| **Foursquare OS Places** ([docs](https://docs.foursquare.com/data-products/docs/fsq-places-open-source)) | 100M+ POIs, website/social attributes | Free | Apache 2.0 |
| **Restaurant websites / PDF menus** | Menus and prices | Tokens only | Respect robots.txt and site terms; record restaurant permission at onboarding; never fetch marketplace domains |
| **Licensed nutrition data** (e.g., Nutritionix) | Chain menu items + nutrition | Commercial | License terms |
| Marketplaces (Uber Eats, PedidosYa, DoorDash) | — | — | Off by default ([ADR-0011](../01-architecture/adr/ADR-0011-marketplace-data-collection.md)) |
| Facebook/Instagram pages | Often list WhatsApp in LATAM | — | Meta terms restrict automated collection → human outreach only |

## 4. The per-client AI research agent ("Neighborhood Scout")

Yes — an AI can investigate restaurants near each client, as a background job when a new delivery place appears.

```text
NeighborhoodScoutWorkflow(placeId)
  1. cell      = H3 cell(s) around the delivery place (radius ~3–5 mi), reuse if scouted < 14 days ago
  2. directory = our merchants + Overture/OSM places in the cells (stored, licensed)            → candidates
  3. live      = Google Places Nearby/Details (Enterprise fields) for top candidates             → hours, open now,
                 price level, rating, business status — used for this client's session, not stored (place ID only)
  4. research  = for candidates without a menu: Claude with server tools
                   web_search (blocked_domains: ubereats.com, doordash.com, grubhub.com, pedidosya.*, rappi.*,
                                facebook.com, instagram.com)
                   web_fetch  (restaurant's own website / PDF menu)
                 → structured output ParsedMenu + wa.me links found + confidence
  5. store     = draft menus with provenance (source=website, trust=medium, expires 30 d) in catalog.merchant_sources
  6. outreach  = queue restaurants with demand but no opt-in for human outreach (call → wa.me JOIN → opt-in)
  7. order     = draft-menu items are proposable but every order is confirmed (items + prices) by the kitchen first
```

### Cost per neighborhood (illustration, assumptions labeled)

| Step | Assumption | Cost |
|---|---|---|
| Google Nearby Search (Enterprise) | 3 calls (20 places each) | 3 × $0.035 = **$0.105** (first 1,000/month free) |
| Claude web search | 15 searches | 15 × $0.01 = **$0.15** |
| Claude web fetch + extraction | 10 menus × ~10K input tokens on Opus 5.5 ($4/MTok in) + ~2K output each ($20/MTok) | ≈ $0.40 + $0.40 = **$0.80** |
| **Total** | per neighborhood scouting, shared by every client there | **≈ $1** (re-scouted every 2–4 weeks) |

Using a cheaper model for menu extraction (e.g., Haiku 5.5) or the Batch API (50 %) can cut step 3 sharply — owner decision after the extraction eval ([AI/11](11-evaluation-and-quality.md)).

### Latency

Scouting runs in the background when the user saves an address; the first session shows verified partner kitchens immediately and "Finding more kitchens near you…" as scouted ones stream in. Live Google data is fetched at request time.

## 5. WhatsApp numbers — what works

- **Phone numbers** are plentiful (88 % / 85 % in Overture). In the US many are landlines; in the Dominican Republic many mobile numbers use WhatsApp, but there's no legitimate way to check WhatsApp registration without contacting the business (unofficial "number checkers" violate WhatsApp's terms — not used).
- **Explicit WhatsApp links** on websites/socials are rare in open data (< 1 %); the research agent also finds `wa.me` links on restaurants' own sites.
- **Opt-in is required** before our WhatsApp Business account sends templates. Flow: human outreach call (AI-drafted script, bilingual) → send `wa.me/<our-number>?text=JOIN%20NB-<code>` by SMS/email/in person → the restaurant's "JOIN" message is the recorded opt-in → templates from then on ([WF/02 §3a](../02-workflows/02-availability-confirmation-whatsapp.md#3a-opt-in-capture-how-a-kitchen-gets-onto-whatsapp-ordering)).
- Restaurants that publicly advertise "order by WhatsApp" invite order messages; a **human** first message asking them to join is reasonable, automated template blasts are not.

## 6. Pilot measurements (decide data strategy with facts)

In the launch zone, track weekly:

| KPI | Target by end of Phase 1 |
|---|---|
| Restaurants in zone (Overture ∪ OSM ∪ Google, deduped, open) | Baseline count |
| With verified phone | ≥ 85 % |
| With draft menu (website/WhatsApp/GBP/POS) | ≥ 50 % |
| With verified menu (kitchen-confirmed) | ≥ 30 % (all partner kitchens) |
| With WhatsApp opt-in | ≥ 25 % (≥ 60 kitchens) |
| Orders where kitchen-confirmed prices differed from draft by > 5 % | Track (drives re-scouting cadence) |

## 7. Data model & code

- `catalog.merchant_sources` (provenance per field, license, trust, expiry) — [data model §5b](../01-architecture/04-data-model.md#5b-cluster-runs--data-provenance-rev-2).
- Adapters: `OvertureAdapter` (monthly Parquet import by metro bbox), `OsmAdapter` (self-hosted Overpass or planet extract), `GooglePlacesAdapter` (runtime only, place-ID storage), `BusinessProfileMenuAdapter` (owner OAuth), `WebsiteMenuAdapter` (Claude web_fetch + vision for PDFs), `WhatsAppMenuAdapter`.
- Dedupe: E.164 phone match, normalized name + distance ≤ 50 m, website domain match; keep source IDs (Overture `id`, Google `place_id`, OSM id).
