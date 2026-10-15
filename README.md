# NothingBetter

**A food concierge you talk to first and order from second.**

NothingBetter works out who is eating, why, and in what mood — using conversation, memory of each person at the table, and specialist AI advisors for nutrition, natural medicine, culture, occasions and social dynamics. It then builds one basket from several restaurants (plus extras from shops on the way), confirms with every kitchen before you pay, and coordinates the delivery.

- **Clients:** Tauri 2 (macOS, Windows, Linux, iOS, Android) + web, one React codebase
- **Backend:** TypeScript (Node 24, Fastify), Temporal workflows, PostgreSQL + PostGIS + pgvector
- **AI:** Claude (Anthropic API) — host agent + council of faculties, structured outputs, rulesets
- **Delivery:** one courier collecting from several nearby kitchens and paying at each counter ("cluster runs", own/partner crew + Onfleet + Stripe Issuing run cards); DoorDash Drive / Uber Direct for single kitchens
- **Restaurant data:** open places data (Overture, OSM) + Google Places at request time + an AI "Neighborhood Scout" reading restaurants' own menus + WhatsApp menus; every order confirmed by the kitchen before payment
- **Integrations:** Stripe (Payments, Connect, Issuing, Tax), DoorDash Drive / Uber Direct, Onfleet, WhatsApp Cloud API, Twilio, Google Places, Mapbox, POS (Square/Toast/Clover)
- **Launch:** one North American metro (recommended: Miami-Dade), then Toronto

## Status

📄 **Planning complete, code not started.** Start with the [documentation index](docs/README.md), then the [MVP backlog](docs/05-delivery-plan/02-mvp-scope-and-backlog.md) (milestone M0).

## Repository layout (planned)

See [monorepo layout](docs/01-architecture/03-monorepo-layout.md). Today the repo contains:

```text
docs/        product, architecture, workflows, AI, design, delivery plan
knowledge/   seed rulesets, cultural & natural-medicine entries, prompts, templates
CLAUDE.md    guidance for coding agents working in this repo
```
