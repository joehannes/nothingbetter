# 02 · Research & Partner Landscape

> Status: Draft v2 · Revised 2026-10-15 · Prices and policies change — re-verify before contracts.
>
> **Revision 2 changes:** (1) corrected the DoorDash Red Card statement — Drive *does* support Dasher-paid purchases in its Shop & Deliver integration; (2) new research on one-trip, multi-kitchen pickup options and the kitchen-cluster radius rule; (3) WhatsApp ordering templates confirmed with opt-in capture; (4) findings on collecting marketplace (Uber Eats, PedidosYa) menu data → [ADR-0011](../01-architecture/adr/ADR-0011-marketplace-data-collection.md).

This document consolidates (a) the earlier ChatGPT investigation ("Food Delivery Platform Assessment", the *restaurant.delivery* concept) and (b) verification of the partners this plan depends on.

## 1. Summary of the earlier investigation (carried over)

The earlier conversation explored a **meta food-delivery platform** ("restaurant.delivery"), piloting in the Dominican Republic (Bávaro/Punta Cana, Santo Domingo). Key conclusions we keep:

1. **The product is a food decision engine**, not another marketplace: *Intent → Context → Constraints → Candidates → Combinations → Ranking → Explanation → Fulfillment*.
2. **Build our own normalized food graph** (restaurants, menus, nutrition, semantics) and attach fulfillment through adapters.
3. **Adapter tiers, best first:** official API → authorized partner API → restaurant's own ordering system → approved browser automation → human-assisted fallback.
4. **Separate permanent preferences from tonight's context.**
5. **Health framing:** nutrition profile + "why it might matter" + healthier alternative; never pseudo-medical claims.
6. **Payments:** the AI never decides to spend; a deterministic validator authorizes exact amounts.
7. **Multi-provider, multi-kitchen ordering is the hardest part.**
8. **Start with food + beverages from shops on the way**, not an arbitrary-errand engine.

Points we **change**:

| Earlier idea | New position | Why |
|---|---|---|
| Company accounts on Uber Eats / PedidosYa operated via headless browsers to **order** | Out of scope | Consumer/API terms prohibit it; account termination risk |
| Collect marketplace **menus/prices/contacts** in the background | Off by default; owner decision with counsel ([ADR-0011](../01-architecture/adr/ADR-0011-marketplace-data-collection.md)) | Terms prohibit it; bot protection; data quality issues (§3.2) |
| Cloudflare Browser Run / Browserbase / Steel as core infra | Optional, only for restaurants' *own* websites with permission | Supply comes from onboarded kitchens |
| DR pilot first | North America first, DR/LATAM as expansion | DoorDash Drive and Uber Direct cover US/Canada |
| Gemini free tier for AI | Claude (Anthropic API) | See [ADR-0004](../01-architecture/adr/ADR-0004-ai-platform-claude.md) |
| One courier picks up everything and pays (founder's model) | **Kept** — as *cluster runs* with our own/partner couriers (Phase 1 pilot) | No courier API offers it turnkey; see §2.2 |

## 2. Delivery

### 2.1 Delivery-as-a-service (DaaS) basics

| Provider | Model | Coverage | Price signal | Notes |
|---|---|---|---|---|
| **DoorDash Drive** | White-label courier; one pickup → one dropoff per delivery | US, Canada, Australia, New Zealand ([FAQ](https://developer.doordash.com/en-US/docs/drive/overview/faqs)) | $9.75 within 5 mi; +$0.75/mi beyond, max 15 mi; $2.75 off if receiver can tip and 100 % passes to DoorDash; returns 60 % of fee ([pricing](https://developer.doordash.com/en-US/docs/drive/overview/pricing_payment)) | Sandbox open; production after terms, demo and review ("update within 5–10 days"). v2 API: single `pickup_address`, `order_route_type: roundtrip` only routing option, no batch endpoints ([API](https://developer.doordash.com/en-US/api/drive/)) |
| **DoorDash Drive — Dasher Shop & Deliver (DSX)** | Dasher shops at *your* store; `order_fulfillment_method: shop_deliver`; `shopping_options.payment_method` `barcode` or `red_card` | As Drive | Not published | **Red Card:** "An amount will be loaded to the Dasher's card at checkout… DoorDash will then invoice the merchant the cost of the items purchased by the Dasher." Enabled via your DoorDash Technical Account Manager ([checkout & audit](https://developer.doordash.com/en-US/docs/drive/how_to/Drive_DSX/how_to_checkout_audit/)). Built on Marketplace-for-Retailers catalog onboarding ([playbook](https://developer.doordash.com/en-US/docs/drive/how_to/Drive_DSX/build_for_dsx/)) |
| **Uber Direct** | Courier for your own orders; quote → create → webhooks | "Available in 2 dozen countries" ([merchant page](https://merchants.ubereats.com/us/en/services/uber-direct/)) | "As low as US$7.99 / CA$9.99 per delivery", no commission | Access "may require written approval" ([dev docs](https://developer.uber.com/docs/deliveries/direct/guides/overview)); one pickup per delivery; webhooks incl. shopping progress; Packing Services (courier collects from merchant shelves) with written authorization; item payment is between merchant and customer |
| **Nash** (orchestrator) | One API to "80+ delivery providers" + your own fleet ([how Nash works](https://docs.usenash.com/reference/how-nash-works)) | US-led | Contract pricing | Dispatch strategies, auto-failover; **batch jobs must share one pickup**: "Batch jobs must share a single pickup location across all packages" ([create batch job](https://docs.usenash.com/api-reference/batch-job/create-batch-job)) |
| **Uber Consumer Delivery API** | Browse Uber Eats, carts, orders inside our app; user links Uber account | Early access, case-by-case ([intro](https://developer.uber.com/docs/consumer-delivery/introduction)) | Not public | Phase 4 supply expansion |
| **PedidosYa** | Partner API (merchant-side); Courier API (logistics) | LATAM | Not public | LATAM expansion |

### 2.2 One trip, many kitchens — options checked 2026-10-15

| Option | Several pickups → one dropoff? | Courier pays at counter? | Where | Verdict |
|---|---|---|---|---|
| **Own/partner crew + dispatch software** — [Onfleet linked tasks](https://support.onfleet.com/hc/en-us/articles/360023669772) ("a set of tasks with a specified order that should be completed by a single driver"), [Shipday](https://www.shipday.com/route-planning) (multi-depot pickups), Nash own-fleet routes | **Yes** | **Yes** (our Stripe Issuing run card) | Any launch zone | **Primary multi-kitchen mode: cluster runs** |
| DoorDash Drive API | No (one pickup per delivery) | Only DSX `red_card`, retail-catalog oriented | US/CA/AU/NZ | Single-kitchen legs; BD ask |
| DoorDash **DoubleDash** (consumer, 2021) | Yes — items from a nearby store, same Dasher, no extra delivery fee ([report](https://www.restaurantbusinessonline.com/technology/new-doordash-feature-doubledash-allows-orders-2-businesses-once)) | n/a | US | Not exposed in Drive → BD ask |
| Uber Eats **multi-store** (consumer, 2023) | Yes — two nearby merchants, one delivery fee ([TechCrunch](https://techcrunch.com/2023/10/11/uber-eats-now-lets-you-order-from-two-nearby-stores-at-the-same-time)) | n/a | US | Not exposed in Direct → BD ask |
| Uber Direct API | No | No | ~24 countries | Fallback provider |
| Nash batch jobs | No (single shared pickup) | No | US | Failover/routing |
| [Roadie](https://www.roadie.com/blog/2022/11/you-asked-we-answered-roadies-most-common-batched-and-routed-delivery-questions-ent) | Batches one origin → many; ~90-min notice | No | 97 % of US households | Scheduled shop-extras only |
| Senpex | Advertises "multi-pickup and multi-drop off options" ([API page](https://web.senpex.com/delivery-service-api)); coverage per third-party listings: SF Bay Area, Seattle, New York | Not stated | Selected metros | Verify in launch city |
| [Dispatch](https://dispatchit.com/customer-faq) (dispatchit) | Multi-stop runs; 80+ markets | Not stated | B2B; no public API found | Verify |
| [Lalamove](https://developers.lalamove.com/) | 1 pickup + up to 15 drop-offs; special requests incl. `HELP_BUY`, `PURCHASE_SERVICE_*` (city-dependent) | Some cities | **Not US**; Mexico City, Brazil, Asia, Middle East | LATAM later |
| **Food halls / multi-brand kitchens** | One address, one consolidated bag | No | Where present | One DaaS leg suffices — prefer in composer |

**Conclusion:** in North America there's no turnkey on-demand courier API that collects from several arbitrary restaurants in one trip and pays at each counter. The founder's model is implemented as **cluster runs** with a courier crew we dispatch (WF/03, WF/08), while we (a) prefer co-located kitchens, (b) keep DaaS single legs and synchronized legs as fallback, and (c) ask DoorDash/Uber business development for multi-pickup and Red Card on Drive/Direct.

### 2.3 Kitchen-cluster radius rule

A basket may mix kitchens only if:
- every kitchen is within **R_cluster ≈ 1.0 km (0.6 mi)** of the first pickup (configurable per metro),
- the extra stops add **≤ 8 min** to the run (routing estimate),
- the cluster is within **5 mi** of the customer (keeps DoorDash's base rate if we fall back to separate legs; Drive adds $0.75/mi beyond 5 mi up to 15 mi — distance surcharges pay for a longer single leg, not for extra pickups),
- co-located kitchens (food hall, plaza, multi-brand kitchen) count as one stop.

### 2.4 Cost comparison (two kitchens)

| Mode | Cost basis | Example |
|---|---|---|
| 2 × DoorDash Drive legs | Published rate | ≥ 2 × $9.75 = **$19.50** + tips |
| 2 × Uber Direct legs | "As low as" | ≥ 2 × $7.99 = **$15.98** + tips |
| 1 cluster run (own/partner crew) | Courier time × all-in hourly cost + vehicle + card fees | *Illustration, assumptions not quotes:* $28/h all-in × 35 min ≈ **$16.30**; at 2 runs/h ≈ **$14** |

Cluster runs win with utilization (≥ 2 runs/h at peak), with 3+ kitchens, and with shop extras. Below that, DaaS legs are cheaper — the dispatcher picks per order (WF/03 §3).

### 2.5 Crew options for cluster runs

| Option | Notes |
|---|---|
| Local courier company on contract | Fastest; insured; negotiate per-run or hourly pricing |
| Shift drivers via staffing platforms | Instawork offers W-2 shifts via an affiliate employer of record (Advantage Workforce Services) per [this profile](https://research.contrary.com/company/instawork); GigSmart lists delivery gigs — verify driver/vehicle/insurance terms |
| Own contractors (Stripe Connect payouts) | Most control; needs labor-law review (city minimum-pay rules, classification) |

## 3. Restaurant supply & data

### 3.1 Sources (recommended)

| Source | Use | Notes |
|---|---|---|
| **Direct onboarding** (partner console, WhatsApp) | Menus (PDF/photo/WhatsApp catalog), hours, prep times, channels, payment method, opt-in | Core of Phase 1: 30–60 kitchens per launch zone |
| **POS integrations** (Square, Toast, Clover) | Live menus, 86'd items, order injection | Restaurant-authorized; Phase 2 |
| **Restaurant's own website / PDF menus** | Menu ingestion | Permission recorded at onboarding |
| **Open places data** — [Overture Maps places](https://docs.overturemaps.org/guides/places/) (~81M places; phones, websites, socials; CDLA-Permissive-2.0/Apache-2.0/CC0), OpenStreetMap (ODbL), [Foursquare OS Places](https://docs.foursquare.com/data-products/docs/fsq-places-open-source) (Apache 2.0) | Stored directory: existence, location, category, phone, website | Measured 2026-10-15: Overture Miami-Dade box 12,553 food places (88 % phone, 79 % website); Gran Santo Domingo 2,286 (85 % phone, 34 % website) — [AI/12](../03-ai/12-restaurant-discovery-and-data-sourcing.md) |
| **Google Places API (New)** | Hours, open-now, price level, phone, website at request time | No menu-items field; no caching except place IDs; Nearby Search Enterprise $35/1,000, Place Details Enterprise $20/1,000 (1,000 free/month each) |
| **AI Neighborhood Scout** | Reads restaurants' own websites/PDF menus per neighborhood (Claude web search $10/1,000 searches; fetch billed as tokens) | ≈ $1 per neighborhood scouting (illustrative); marketplace and Meta domains blocked |
| **Google Business Profile API** (`getFoodMenus`) | The restaurant's own Google menu | Owner OAuth consent at onboarding |
| **Licensed nutrition data** — e.g., [Nutritionix](https://nutritionix.com/business) (restaurant items, database licensing) | Nutrition for chain items | Commercial license |

### 3.2 Marketplace data (Uber Eats, PedidosYa, DoorDash) — findings

| Topic | Finding |
|---|---|
| Uber US terms | "You may not use the Services, or any content available through the Services, **whether or not you are logged in**, for any commercial purpose or for developing software, including training machine-learning or AI systems, without Uber's prior written permission." Also: services "made available solely for your noncommercial use"; "By accessing or using the Services, you confirm your agreement to be bound" ([terms](https://www.uber.com/legal/en/document/?name=general-terms-of-use&country=united-states&lang=en)) |
| PedidosYa DR terms | "Queda prohibido modificar, copiar, reutilizar, extraer, explotar, reproducir, comunicar al público… la totalidad o parte de los contenidos incluidos en el Portal. El uso no autorizado del Material puede constituir una violación de las leyes… sobre derechos de autor, leyes de propiedad industrial u otras leyes." ([DR terms](https://terms.pystatic.com/about/generales/tyc_generales_dominican_republic.html)) |
| Technical | 2026-10-15: plain HTTP requests to ubereats.com and doordash.com (even `/robots.txt`) received a Cloudflare challenge; pedidosya.com.do/.uy/.ar returned "Access to this page has been denied" |
| US case law | Public-page scraping is generally not a CFAA violation (*hiQ v. LinkedIn*, 9th Cir.); *Meta v. Bright Data* (N.D. Cal., Jan 23 2024): Meta's then-terms didn't bind **logged-out** scrapers ([analysis](https://www.fbm.com/business-litigation/publications/major-decision-affects-law-of-scraping-and-online-data-collection-meta-platforms-v-bright-data/)). Uber's current terms expressly reach logged-out commercial use → contract/tort claims, cease-and-desist, IP blocking remain realistic. Bare facts (dish names, prices) get little copyright protection; photos and creative descriptions are protected |
| Data quality | Uber Eats doesn't publish restaurant phone numbers (no WhatsApp numbers to harvest). App totals ran ~66–71 % above in-store for the same McDonald's order in 2025, counting fees, tax and a 15 % tip ([Self study](https://www.self.inc/info/most-expensive-cities-for-food-delivery-apps/)) — listed app prices are not what a kitchen charges us directly |
| WhatsApp | Harvested numbers don't satisfy WhatsApp's opt-in rule ("You may only contact people on WhatsApp if… they have given you their mobile phone number… and you have received opt-in permission") |

Decision record: [ADR-0011](../01-architecture/adr/ADR-0011-marketplace-data-collection.md) (owner decision; default off).

## 4. Messaging: WhatsApp, SMS, voice

- **Ordering via WhatsApp templates — confirmed approach.** Utility templates for availability check, order confirmation (with allergy line), courier-pays-at-counter notice, ready ping, cancellation (EN/ES). See [WF/02](../02-workflows/02-availability-confirmation-whatsapp.md).
- **Pricing:** since **July 1, 2025** per delivered template; non-template messages inside an open 24-h window are free; utility templates inside the window are free ([pricing](https://developers.facebook.com/docs/whatsapp/pricing/)). Rate updates for several markets effective **Oct 1, 2026** — re-check US/CA.
- **Opt-in:** required before business-initiated messages; separate opt-in per category is best practice; honor opt-outs; automation needs human escalation ([policy](https://whatsappbusiness.com/policy/)). **Capture:** at onboarding the kitchen messages us first (wa.me link or QR from a call/visit), or ticks an opt-in box in the partner console.
- **AI chatbot ban (from Jan 15, 2026):** targets general-purpose assistants; business-specific order workflows remain allowed ([summary](https://respond.io/blog/whatsapp-general-purpose-chatbots-ban)).
- **North America:** many restaurants answer SMS/phone faster → multi-channel liaison remains. SMS needs A2P 10DLC (US) / CASL (Canada); AI voice calls need consent (FCC 2024 TCPA ruling).

## 5. Payments

- **Stripe Payments** (manual capture), **Stripe Issuing** (single-use merchant cards; **courier run cards** for cluster runs — spending controls by category, card presence, per-authorization/daily limits; defaults 500 USD/day per new card and a 10,000 USD per-authorization ceiling unless changed — [spending controls](https://docs.stripe.com/issuing/controls/spending-controls)), **Stripe Connect** (partner settlement, courier payouts), **Stripe Tax**.

## 6. AI platform

Claude via the Anthropic API — see [AI/01](../03-ai/01-ai-platform-architecture.md).

## 7. Competitive notes

Marketplaces optimize their own networks; restaurant-side aggregators (Deliverect, Chowly, KitchenHub, Klikit, Olo) serve operators; Google hands off to providers. The gap remains a consumer-side food decision engine that coordinates several kitchens into one delivery.

## 8. Verification log

| Claim | Source opened | Date |
|---|---|---|
| DoorDash Drive countries, production review | developer.doordash.com FAQ | 2026-10-15 |
| DoorDash Drive pricing | developer.doordash.com pricing & payment | 2026-10-15 |
| Drive v2 single pickup; roundtrip only; no batch endpoints | developer.doordash.com/en-US/api/drive | 2026-10-15 |
| Drive DSX red_card & invoicing | developer.doordash.com DSX checkout & playbook | 2026-10-15 |
| Uber Direct pricing & regions | merchants.ubereats.com | 2026-10-15 |
| Uber Direct webhooks & approval | developer.uber.com | 2026-10-15 |
| Nash batch jobs single pickup | docs.usenash.com create batch job | 2026-10-15 |
| Lalamove stops, markets, special requests | developers.lalamove.com | 2026-10-15 |
| Senpex multi-pickup claim | web.senpex.com | 2026-10-15 |
| Roadie batching model | roadie.com blog | 2026-10-15 |
| Uber terms (logged-out commercial use) | uber.com legal | 2026-10-15 |
| PedidosYa DR terms (extraction ban) | terms.pystatic.com | 2026-10-15 |
| Meta v. Bright Data | fbm.com analysis | 2026-10-15 |
| Delivery-app markup study | self.inc | 2026-10-15 |
| WhatsApp pricing, opt-in, escalation | developers.facebook.com, whatsappbusiness.com | 2026-10-15 |
| Stripe Issuing spending controls | docs.stripe.com | 2026-10-15 |
| Tauri geolocation permissions | v2.tauri.app | 2026-10-15 |
