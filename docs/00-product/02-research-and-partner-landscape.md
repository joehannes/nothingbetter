# 02 · Research & Partner Landscape

> Status: Draft v1 · As of 2026-10-15 · Prices and policies change — re-verify before contracts.

This document consolidates (a) the earlier ChatGPT investigation ("Food Delivery Platform Assessment", the *restaurant.delivery* concept) and (b) fresh verification of the partners this plan depends on.

## 1. Summary of the earlier investigation (carried over)

The earlier conversation explored a **meta food-delivery platform** ("restaurant.delivery"), piloting in the Dominican Republic (Bávaro/Punta Cana, Santo Domingo). Key conclusions we keep:

1. **The product is a food decision engine**, not another marketplace: *Intent → Context → Constraints → Candidates → Combinations → Ranking → Explanation → Fulfillment*.
2. **Build our own normalized food graph** (restaurants, menus, nutrition, semantics) and attach fulfillment through adapters.
3. **Adapter tiers, best first:** official API → authorized partner API → restaurant's own ordering system → approved browser automation → human-assisted fallback.
4. **Separate permanent preferences from tonight's context** ("I ate badly yesterday, want something fresh" must not overwrite "likes burgers").
5. **Health framing:** nutrition profile + "why it might matter" + healthier alternative; never pseudo-medical claims.
6. **Payments:** the AI never decides to spend; a deterministic validator authorizes exact amounts (restaurant ✓ items ✓ total ✓ address ✓ budget ✓).
7. **Multi-provider, multi-kitchen ordering is the hardest part** (two kitchens = two orders, two ETAs, possibly two couriers).
8. **Don't build the "arbitrary errand" engine yet** — start with food + beverages from shops on the way.

Points we **change**:

| Earlier idea | New position | Why |
|---|---|---|
| Use company accounts on Uber Eats / PedidosYa via headless browsers | **Out of scope** unless a provider authorizes it in writing | Their consumer and API terms prohibit scraping and automated ordering; account termination risk; the same investigation flagged this |
| Cloudflare Browser Run / Browserbase / Steel as core infra | **Optional** — only for crawling restaurants' *own* websites with permission | Our supply comes from onboarded restaurants and POS integrations |
| DR pilot first | **North America first**, DR/LATAM as expansion | DoorDash Drive and Uber Direct both cover US/Canada; user request |
| Gemini free tier for AI | **Claude (Anthropic API)** as the AI platform | Quality of multi-faculty reasoning, tool use, structured outputs; see [ADR-0004](../01-architecture/adr/ADR-0004-ai-platform-claude.md) |

## 2. Delivery-as-a-service (DaaS) options

| Provider | Model | Coverage | Price signal | Notes |
|---|---|---|---|---|
| **DoorDash Drive** | White-label courier; one pickup → one dropoff | US, Canada, Australia, New Zealand ([FAQ](https://developer.doordash.com/en-US/docs/drive/overview/faqs)) | $9.75 within 5 mi; +$0.75/mi beyond, max 15 mi; $2.75 discount if receiver can tip and 100 % passes to DoorDash; returns 60 % of fee ([pricing](https://developer.doordash.com/en-US/docs/drive/overview/pricing_payment)) | Sandbox open to anyone; production after acknowledging terms, a demo and review, "update within 5–10 days". Billed per delivery to card, invoice possible. |
| **Uber Direct** | Courier for your own orders; quote → create → webhooks | "Available in 2 dozen countries" ([merchant page](https://merchants.ubereats.com/us/en/services/uber-direct/)) | "As low as US$7.99 / CA$9.99 per delivery", no commission, no minimums | API access "may require written approval" ([dev docs](https://developer.uber.com/docs/deliveries/direct/guides/overview)); webhooks: delivery status, courier update, refund request, shopping progress; Courier Pick & Pack API exists. Alcohol "may be restricted depending on your geography". |
| **Nash** (orchestrator) | One API to "80+ delivery providers" + your own fleet ([how Nash works](https://docs.usenash.com/reference/how-nash-works)) | US-led | Contract pricing | Dispatch strategies (price, preferred order, reliability), auto-failover, webhooks for status/ETA/proof of delivery; advertises "multi-stop, and batched deliveries across 80+ providers and your own fleet" ([orchestration](https://docs.usenash.com/reference/orchestration)). |
| **Own / partner courier pool** | Our dispatch (e.g., Onfleet, Nash own-fleet routes) | Launch zone only | Per run / per hour | Required for one-courier multi-pickup with shop purchases. |
| **Uber Consumer Delivery API** | Browse Uber Eats, carts, orders inside our app; user links Uber account | Early access, case-by-case ([intro](https://developer.uber.com/docs/consumer-delivery/introduction)) | Not public | Explicitly lists AI-powered platforms/chatbots as use cases. Phase 4 supply expansion. |
| **DoorDash Marketplace API** | Restaurant/menu/order | Restricted, selected partners | — | Not for MVP. |
| **PedidosYa** | Partner API (merchant-side); Courier API (logistics) | LATAM | Not public | LATAM expansion: Courier API is the DaaS analogue. |
| **Deliveroo Signature, JET Go, Wolt Drive** | Custom experience / DaaS | EU/UK/others | Contract | Later geographies. |

### Critical finding: couriers don't pay restaurants

- Neither DoorDash Drive nor Uber Direct documents a **multi-pickup, single-dropoff** delivery in one job; the core model is one pickup → one dropoff.
- DaaS couriers pick up **prepaid, ready** orders. DoorDash's **Red Card** (a prepaid card Dashers use to buy items) is only for DoorDash's own *Shop & Deliver* orders ([Dasher blog](https://dasher.doordash.com/en-us/blog/what-is-a-doordash-red-card)), not a Drive API feature.
- **Consequence:** NothingBetter pays restaurants directly (partner settlement or single-use virtual card). Courier-paid purchases only happen in our own/partner courier pool, with a capped Stripe Issuing card. See [WF/04 Payments](../02-workflows/04-payments-and-money-flow.md).

## 3. Restaurant supply (North America, without scraping)

| Source | Use | Notes |
|---|---|---|
| **Direct onboarding** (partner console) | Menus, hours, prep times, channels, payment method | Core of Phase 1: 30–60 curated restaurants per launch zone |
| **POS integrations** (Square, Toast, Clover) | Live menus, 86'd items, order injection | Restaurant authorizes via OAuth / partner program; Phase 2 |
| **Restaurant's own website / PDF / photo menus** | Menu ingestion with permission | Claude vision extraction; human review queue |
| **Places data** (e.g., Foursquare OS Places, Google Places) | Location, hours, categories for discovery & outreach | Respect license terms (Google restricts caching/mixing) |
| Marketplaces' consumer apps | **Not used** | ToS prohibit automated access |

## 4. Messaging: WhatsApp, SMS, voice

- **Pricing model:** since **July 1, 2025** WhatsApp charges **per delivered template message**; non-template messages inside an open 24-hour customer service window are free; utility templates sent inside the window are free ([pricing](https://developers.facebook.com/docs/whatsapp/pricing/)). Rates may change only on the first day of a quarter; new rate updates are effective **October 1, 2026** for several markets — re-check the US/CA rate card.
- **Window rules:** a window opens when the recipient messages us and closes 24 h after their last message; outside it only approved templates.
- **Opt-in:** you may only contact people who gave you their number **and** opted in; separate opt-in by message category is best practice; honor opt-outs ([policy](https://whatsappbusiness.com/policy/)).
- **Automation:** allowed in the window but "must also have available prompt, clear, and direct escalation paths" (human transfer, phone, email…).
- **AI chatbot ban:** from **January 15, 2026** Meta prohibits *general-purpose* AI assistants on the Business API; business-specific bots (orders, support, notifications) remain allowed ([summary](https://respond.io/blog/whatsapp-general-purpose-chatbots-ban)). Our restaurant liaison and any consumer WhatsApp channel must stay food-ordering-specific.
- **AI Providers pricing** (non-template messages charged) applies only to AI providers in specific mandated markets (Italy, EU, Brazil); not to us as a food business. Reported plans for token-based business-agent charges from Aug 2026 are unverified — track.
- **North American reality:** many US/CA restaurants respond to **SMS** or a **phone call** faster than WhatsApp. WhatsApp is strong with Latino, South Asian and other diaspora-owned restaurants (relevant for Miami, Houston, Toronto). Hence a multi-channel liaison ([WF/02](../02-workflows/02-availability-confirmation-whatsapp.md)).
- **SMS (US):** A2P 10DLC brand/campaign registration required; **Canada:** CASL consent rules.
- **AI voice calls:** the FCC treats AI-generated voices as "artificial" under the TCPA (2024 ruling) → obtain consent during restaurant onboarding before automated calls.

## 5. Payments

- **Stripe Payments** with manual capture (authorize at checkout, capture at fulfillment) and Apple Pay / Google Pay.
- **Stripe Issuing** for single-use virtual cards and courier cards: spending controls by merchant category, country, card presence, per-authorization / daily / monthly limits; defaults of 500 USD/day per new card and a 10,000 USD per-authorization ceiling apply unless changed ([spending controls](https://docs.stripe.com/issuing/controls/spending-controls)); real-time authorization webhooks for approve/decline.
- **Stripe Connect** to settle partner restaurants and pay own couriers.
- **Sales tax:** as the seller/marketplace facilitator of prepared food we collect and remit sales tax → Stripe Tax or Avalara.

## 6. AI platform

Claude via the Anthropic API (TypeScript SDK `@anthropic-ai/sdk`). Current models (cached 2026-10-06): Opus 5.5 `claude-opus-5-5` ($4/$20 per MTok), Sonnet 5.5 `claude-sonnet-5-5` ($2/$10), Haiku 5.5 `claude-haiku-5-5` ($0.10/$0.50 up to 100K-token prompts). Details and API gotchas in [AI/01](../03-ai/01-ai-platform-architecture.md).

## 7. Competitive notes

- **Marketplaces** (Uber Eats, DoorDash, Grubhub): catalog-first, single-restaurant baskets; their AI features optimize their own network.
- **Restaurant-side aggregators** (Deliverect, Chowly, KitchenHub, Klikit, Olo): connect restaurants to many marketplaces — prove that normalization across providers is viable, but serve the operator, not the eater.
- **Google** shows ordering providers on Maps/Search but hands off to them.
- **Gap:** "Tell me what's happening in your life tonight and I'll figure out what food makes sense — for everyone at the table — and get it here together." No major player does that end to end.

## 8. Verification log

| Claim | Source opened | Date |
|---|---|---|
| DoorDash Drive countries, production review | developer.doordash.com FAQ | 2026-10-15 |
| DoorDash Drive pricing | developer.doordash.com pricing & payment | 2026-10-15 |
| Uber Direct pricing & regions | merchants.ubereats.com Uber Direct | 2026-10-15 |
| Uber Direct webhooks & approval | developer.uber.com Direct overview | 2026-10-15 |
| Nash providers, dispatch, batching | docs.usenash.com | 2026-10-15 |
| WhatsApp per-message pricing & windows | developers.facebook.com pricing | 2026-10-15 |
| WhatsApp opt-in & escalation | whatsappbusiness.com/policy | 2026-10-15 |
| Stripe Issuing spending controls | docs.stripe.com | 2026-10-15 |
| Red Card scope | dasher.doordash.com | 2026-10-15 |
| Tauri geolocation plugin permissions | v2.tauri.app/plugin/geolocation | 2026-10-15 |
