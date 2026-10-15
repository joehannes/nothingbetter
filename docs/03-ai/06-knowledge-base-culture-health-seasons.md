# AI/06 · Knowledge Base — Culture, Health, Natural Medicine, Seasons, Occasions

> Status: Draft v1 · Storage: `knowledge.kn_*` tables, authored as YAML in `knowledge/` (git-versioned, schema-validated, expert-reviewed).

## 1. Purpose

Faculties reason better when grounded in curated knowledge rather than model memory alone. The knowledge base gives them **facts with sources, licenses, evidence levels and reviewers**, and gives the product a defensible, improvable asset.

## 2. Domains

| Domain | Entities | Examples |
|---|---|---|
| **Cuisines & dishes** | cuisine, canonical dish, regional variant, technique | `dish:ropa_vieja` (Cuban), `dish:khichdi`, `dish:pho_bo`, `technique:wok_hei` |
| **Ingredients & nutrition** | ingredient, nutrient profile, allergen class | `ingredient:sesame` → allergen, `ingredient:ginger` → nutrients |
| **Cultures & heritage** | culture/heritage group, eating style, comfort foods, language names | `culture:cuban`, `culture:punjabi`, `culture:ethiopian`, `culture:quebecois`, `culture:soul_food_tradition` |
| **Religious & ethical rules** | rule set, permitted/forbidden, certification types | halal, kosher/kosher-style, Hindu vegetarian, Jain (no root vegetables), Lent Fridays, Buddhist vegetarian days, vegan |
| **Occasions & calendar** | occasion, festival, observance, typical foods, timing rules | Thanksgiving (US: 4th Thu Nov; Canada: 2nd Mon Oct), Lunar New Year, Ramadan/iftar (sunset), Eid, Diwali, Passover, Hanukkah, Christmas/Nochebuena, Día de Muertos, Super Bowl, Canada Day, July 4th, Juneteenth, Nowruz, Mother's/Father's Day, birthdays |
| **Seasons & produce** | season, region, local produce availability | Florida citrus winter peak, Ontario autumn squash/apples, summer stone fruit |
| **Traditional medicine** | TCM nature/flavor, Ayurveda rasa/virya/dosha effects, folk remedies, herb profiles | ginger = warming; cucumber = cooling; khichdi = tridoshic comfort food |
| **Health guidance** | condition patterns, life stages, goals, interactions | Lower-sodium pattern for hypertension; pregnancy food safety; grapefruit interactions |
| **Social & etiquette** | serving styles, hosting norms, gift foods, condolence foods | Family-style sharing, bringing dessert, foods for new parents |
| **Regional context (supply-side)** | metro food scene, popular cuisines, local specialties | Miami: Cuban, Haitian, Venezuelan, Peruvian; Toronto: Caribbean, South Asian, Chinese, Portuguese |

## 3. Entry format (YAML)

```yaml
# knowledge/cultures/cuban.yaml
code: culture:cuban
type: culture
labels: { en: Cuban, es: Cubana }
locale_scope: [US-FL, US-NJ, US-NY]
eating_style: family-style dinners, late Sunday lunches, strong coffee rituals
comfort_dishes:
  - dish:ropa_vieja
  - dish:arroz_con_pollo
  - dish:picadillo
  - dish:sopa_de_pollo
celebrations:
  - occasion: nochebuena
    foods: [dish:lechon_asado, dish:moros_y_cristianos, dish:yuca_con_mojo, dish:turron]
flavor_signature: [cumin, oregano, garlic, sour orange, sofrito]
bridges:            # respectful discovery suggestions for curious users
  - { to: culture:puerto_rican, via: [dish:pernil, dish:mofongo] }
  - { to: culture:peruvian, via: [dish:lomo_saltado] }
cautions:
  - "Never assume a Spanish-speaking user is Cuban; use only if self-declared."
sources: [src:wikidata_cuisine_cuba, src:culinary_consultant_review_2026_10]
review: { reviewer_role: cultural_consultant, reviewed_at: 2026-10-10, next_review: 2027-04-10 }
version: 1
```

```yaml
# knowledge/natural-medicine/ginger.yaml
code: ingredient:ginger
type: ingredient
traditional:
  tcm: { nature: warming, flavor: [pungent], uses: ["dispels cold", "settles the stomach"] }
  ayurveda: { rasa: [katu], virya: ushna, dosha: { vata: decreases, kapha: decreases, pitta: increases_in_excess } }
  folk: ["ginger tea for colds", "ginger for travel sickness"]
evidence:
  - claim: "May reduce nausea (pregnancy, post-operative, motion)"
    level: B
    source: src:nccih_ginger
interactions:
  - with: anticoagulants
    severity: caution
    note: "High supplemental doses may increase bleeding risk; culinary amounts generally considered safe."
    source: src:msk_about_herbs_ginger
phrasing:
  en: "Ginger has a long tradition for settling the stomach, and studies support a modest effect on nausea."
review: { reviewer_role: licensed_naturopath_and_RD, reviewed_at: 2026-10-12, next_review: 2027-04-12 }
version: 1
```

JSON Schemas for each type live in `packages/knowledge-kit/schemas/` and are enforced in CI.

## 4. Evidence levels (health & natural medicine)

| Level | Meaning | Allowed phrasing |
|---|---|---|
| **A** | Strong evidence (multiple quality RCTs/meta-analyses, guideline-endorsed) | "Research shows…" |
| **B** | Moderate/limited clinical evidence | "Studies suggest… / may help…" |
| **C** | Traditional use, mechanistic or weak evidence | "Traditionally used for… / many people find…" |
| **D** | Folk belief, no supporting evidence or conflicting | Only as cultural story, never as benefit claim |

Never: "cures", "treats", "prevents [disease]", dosage advice, replacing medication.

## 5. Sources (candidate, check license before ingest)

| Source | Content | License / terms note |
|---|---|---|
| USDA FoodData Central | Nutrient data | Public domain (CC0) |
| Canadian Nutrient File (Health Canada) | Nutrient data | Open Government Licence – Canada |
| Open Food Facts | Packaged products, allergens | ODbL (share-alike on database) — isolate if used |
| FDA / Health Canada allergen lists | Major/priority allergens | Public |
| Dietary Guidelines for Americans; Canada's Food Guide | Dietary patterns | Public |
| NIH Office of Dietary Supplements; NCCIH | Herbs/supplements evidence | Public (US gov) |
| Memorial Sloan Kettering "About Herbs" | Herb profiles, interactions | Check terms; reference/linking |
| Wikidata | Cuisines, dishes, festivals, multilingual labels | CC0 |
| Hebcal (Jewish calendar), astronomical sunset computation | Observance timing | Check API terms |
| National statistics (US Census ACS, Statistics Canada) | Regional demographics for **supply-side** planning only | Public |
| Expert consultants (RD, licensed naturopath / TCM practitioner, cultural consultants per heritage) | Authoring & review | Contracted |

## 6. Governance

- **Authoring:** PRs to `knowledge/`; CI validates schema, sources, and labels; health/natural-medicine changes require the `review:health` label by a credentialed reviewer; cultural packs require `review:culture` by a consultant from that community.
- **Versioning:** every entry has `version`; faculties' knowledge packs record the knowledge commit hash in `ai_runs`.
- **Refresh:** review dates; yearly recompute of moving festivals (lunar calendars) into `occasion_calendar`.
- **Bias checks:** quarterly audit of suggestions by declared heritage for stereotyping; community feedback channel.

## 7. Region packs (North America, launch)

| Pack | Contents |
|---|---|
| `region:us-fl-miami` | Local cuisines (Cuban, Haitian, Venezuelan, Colombian, Peruvian, Nicaraguan, Puerto Rican, Jamaican, Southern US), seasonal produce, hurricane-season comfort patterns, bilingual EN/ES naming |
| `region:ca-on-toronto` | Caribbean, South Asian, Chinese, Portuguese, Korean, Ethiopian, Middle Eastern; Canadian holidays; EN/FR labels |
| `region:us-ny-nyc`, `region:us-tx-houston`, `region:us-il-chicago`, `region:ca-bc-vancouver` | Phase 3+ |

## 8. How faculties use the knowledge

- Knowledge packs are **compiled per faculty** (relevant subset, compact text) and cached as a prompt prefix (`cache_control`).
- For specific dishes/ingredients, faculties call a **knowledge lookup** at prompt-build time (not as a model tool) to keep latency low.
- Faculty outputs must cite `kn` codes for health and cultural claims; the UI's "Why this?" renders the citations.
