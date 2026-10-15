# NothingBetter — Documentation Index

> Status: Draft v2 · 2026-10-15 (rev 2: cluster runs, WhatsApp ordering, marketplace data, restaurant discovery) · The plan for a Tauri 2 + React food-concierge platform (desktop, mobile, web) with AI-led, socially intelligent ordering from several restaurants and one coordinated delivery.

## Reading order

1. **Product** — why and for whom
   - [01 Vision](00-product/01-vision.md)
   - [02 Research & partner landscape](00-product/02-research-and-partner-landscape.md) (incl. summary of the earlier ChatGPT investigation)
   - [03 Personas & journeys](00-product/03-personas-and-journeys.md)
   - [04 Glossary](00-product/04-glossary.md)
2. **Architecture** — how it's built
   - [01 System architecture](01-architecture/01-system-architecture.md)
   - [02 Tech stack](01-architecture/02-tech-stack.md)
   - [03 Monorepo layout](01-architecture/03-monorepo-layout.md)
   - [04 Data model](01-architecture/04-data-model.md)
   - [05 API & events](01-architecture/05-api-and-events.md)
   - [06 Security, privacy & compliance](01-architecture/06-security-privacy-compliance.md)
   - [07 Infrastructure & ops](01-architecture/07-infrastructure-and-ops.md)
   - [ADRs](01-architecture/adr/) — 0001 Tauri+React · 0002 Postgres · 0003 Modular monolith + Temporal · 0004 Claude · 0005 Delivery orchestration · 0006 Merchant of record · 0007 Multi-channel liaison · 0008 Check-then-pay · 0009 Voice pipeline · 0010 No sensitive inference · 0011 Marketplace data collection (proposed)
3. **Workflows** — how an order moves
   - [01 Order lifecycle](02-workflows/01-order-lifecycle.md)
   - [02 Availability & confirmation (WhatsApp automation)](02-workflows/02-availability-confirmation-whatsapp.md)
   - [03 Dispatch, multi-pickup & shop extras](02-workflows/03-dispatch-multi-pickup-and-shop-extras.md)
   - [04 Payments & money flow](02-workflows/04-payments-and-money-flow.md)
   - [05 Location, GPS & addresses](02-workflows/05-location-gps-and-addresses.md)
   - [06 Merchant onboarding & partner console](02-workflows/06-merchant-onboarding-and-partner-console.md)
   - [07 Support, refunds & exceptions](02-workflows/07-support-refunds-and-exceptions.md)
   - [08 Courier operations (cluster runs)](02-workflows/08-courier-operations.md)
4. **AI** — the brain
   - [01 AI platform architecture](03-ai/01-ai-platform-architecture.md)
   - [02 AI faculties & skillsets](03-ai/02-ai-faculties-skillsets.md)
   - [03 Assistive skillsets (human desirables)](03-ai/03-assistive-skillsets.md)
   - [04 Conversation engine & adaptive dials](03-ai/04-conversation-engine-and-adaptive-dials.md)
   - [05 Avatars, memory & personalization](03-ai/05-memory-avatars-and-personalization.md)
   - [06 Knowledge base: culture, health, seasons](03-ai/06-knowledge-base-culture-health-seasons.md)
   - [07 Background ingestion & parsing](03-ai/07-background-ingestion-and-parsing.md)
   - [08 Voice & speech](03-ai/08-voice-and-speech.md)
   - [09 Safety & health guardrails](03-ai/09-safety-and-health-guardrails.md)
   - [10 Prompts, rulesets & templates](03-ai/10-prompts-rulesets-and-templates.md)
   - [11 Evaluation & quality](03-ai/11-evaluation-and-quality.md)
   - [12 Restaurant discovery & data sourcing (Neighborhood Scout)](03-ai/12-restaurant-discovery-and-data-sourcing.md)
5. **Design** — how it looks, sounds and feels
   - [01 Principles & brand](04-design/01-design-principles-and-brand.md)
   - [02 Design system](04-design/02-design-system.md)
   - [03 Screens & flows](04-design/03-screens-and-flows.md)
   - [04 Voice UX & conversation design](04-design/04-voice-ux-and-conversation-design.md)
   - [05 Accessibility & i18n](04-design/05-accessibility-and-i18n.md)
6. **Delivery plan** — how we get there
   - [01 Roadmap](05-delivery-plan/01-roadmap.md)
   - [02 MVP scope & backlog](05-delivery-plan/02-mvp-scope-and-backlog.md) ← start coding here
   - [03 Testing strategy](05-delivery-plan/03-testing-strategy.md)
   - [04 Risks & open questions](05-delivery-plan/04-risks-and-open-questions.md)
   - [05 Launch checklist](05-delivery-plan/05-launch-checklist.md)

Seed knowledge, rulesets and prompts: [`/knowledge`](../knowledge/README.md).

## Requirement → document map

| Request | Where |
|---|---|
| Architectural docs | `01-architecture/*`, ADRs |
| Design docs | `04-design/*` |
| Workflow docs | `02-workflows/*` |
| Skillset / assistive skillset / AI skillset docs | `03-ai/02`, `03-ai/03` |
| WhatsApp automation | `02-workflows/02`, templates in `knowledge/templates/merchant/` |
| Background AI platform parsing automation | `03-ai/07` |
| Voice recognition / speech | `03-ai/08`, `04-design/04` |
| Live adaptation of urgency, inspiration, directness, scenicness | `03-ai/04`, `knowledge/rulesets/dials/signals.yaml` |
| Avatar system | `03-ai/05` |
| Cultural & health knowledge DB | `03-ai/06`, `knowledge/cultures`, `knowledge/natural-medicine`, `knowledge/calendar` |
| GPS integration | `02-workflows/05` |
| Payments, delivery-service integration (DoorDash etc.) | `02-workflows/04`, `02-workflows/03`, `00-product/02` |
| One courier, several kitchens, courier pays (cluster runs) | `02-workflows/03`, `02-workflows/08`, ADR-0005/0006 |
| Restaurant DB from Google Maps/open data, AI research per client | `03-ai/12`, ADR-0011 |
| Marketplace (Uber Eats/PedidosYa) data | `00-product/02` §3.2, ADR-0011 |

## Conventions

- Facts about third parties carry an "as of" date and links; re-verify before contracts.
- ⚖️ marks items needing legal review.
- Decisions go into ADRs; open decisions into [risks & open questions](05-delivery-plan/04-risks-and-open-questions.md).
