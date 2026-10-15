# WF/06 · Merchant Onboarding & Partner Console

> Status: Draft v1 · Phase 1 supply: 30–60 curated restaurants + 10–20 shops in one launch zone.

## 1. Merchant tiers

| Tier | Agreement | Payment | Channels | Data quality |
|---|---|---|---|---|
| **Partner** | Signed terms (prepaid courier pickups, response SLAs, allergen declarations) | Stripe Connect settlement | POS or WhatsApp/SMS + partner console | Merchant-reviewed menu & allergens |
| **Listed** | Light agreement / verbal + opt-in to messages | Single-use virtual card per order | WhatsApp/SMS/voice | AI-parsed menu, merchant spot-checked |
| **Prospect** | None | — | — | Not orderable |

## 2. Onboarding flow (partner console, ~15 minutes)

1. **Business basics:** name, address(es), hours, cuisine, languages, contact person.
2. **Channels & opt-in:** WhatsApp number (recommended), SMS number, phone for calls, quiet hours; explicit opt-in checkboxes with policy text (recorded with timestamp/version).
3. **Menu:** connect POS (Square/Toast/Clover) **or** upload PDF/photos/URL → AI parse → review screen ("What we understood") → fix items, mark sold-out items, declare labels/allergens/certifications (halal, kosher, gluten-free kitchen).
4. **Operations:** average prep time by daypart, max concurrent pickup orders, pickup instructions (counter, side door), packaging notes.
5. **Payments:** Connect onboarding (partner) or virtual-card acceptance method (listed): phone card entry, online ordering link, or in-person card payment by our courier at pickup (cluster run).
6. **Allergen & cross-contact policy:** short questionnaire; can they guarantee no sesame/peanut contact? (Needed to serve severe-allergy seats.)
7. **Go live:** test order with simulator; status → active.

Field onboarding: ops visits with a tablet for the first 30 merchants (relationship + menu photos + opt-in).

## 3. Partner console features

| Feature | Phase |
|---|---|
| Live order board (checks, confirmed, ready, picked up) with sound alerts | 1 |
| One-tap respond to checks (alternative to WhatsApp) | 1 |
| Menu editor with AI suggestions, 86 toggles | 1 |
| Payouts & statements | 1 |
| Performance: answer times, acceptance, ratings by dish (anonymized) | 2 |
| Demand insights: "Customers nearby asked for vegan Ethiopian 34 times this week" | 2 |
| Promotions (opt-in, honest) | 3 |

## 4. Merchant terms (key points to draft) ⚖️

- Orders are **prepaid by NothingBetter**; courier picks up; merchant prepares by `ready_by`.
- Response SLA for checks (e.g., answer within 5 min during open hours) — best effort for listed tier.
- Allergen declarations are the merchant's responsibility; NothingBetter shows them as provided.
- Menu prices on NothingBetter equal in-store/online prices (or declared markup).
- Messaging consent and opt-out mechanics.
- Data use: we don't sell merchant data; demand insights aggregated.

## 5. Shops for extras

Same flow, simplified: categories carried (drinks, flowers, bakery, grocery basics), hours, whether they accept courier card payments, optional catalog via POS. No alcohol in v1.
