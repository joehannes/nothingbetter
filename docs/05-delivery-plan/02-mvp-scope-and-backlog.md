# Plan/02 · MVP Scope & Backlog

> Status: Draft v1 · Phase 1 backlog, ordered. Each story has acceptance criteria (AC) so implementation can start directly. IDs are stable references for PRs (`NB-M3-04`).

## Scope guardrails (Phase 1)

**In:** web + desktop app, EN/ES, avatars/households/memory, council, proposals & Table view, single-restaurant orders via DaaS, **cluster-run pilot (2–3 nearby kitchens, one courier paying at each counter) in one zone**, WhatsApp ordering templates + SMS, Stripe payments + Issuing run cards, push-to-talk voice, partner console v0, ops console v0.
**Out (later):** shop extras at scale, synchronized DaaS legs, POS, mobile builds, hands-free voice (Phase 2); calendar integrations, second metro (Phase 3); alcohol; marketplace data collection (ADR-0011 default off).

---

## M0 · Foundations

| ID | Story | AC |
|---|---|---|
| NB-M0-01 | Scaffold monorepo per [layout](../01-architecture/03-monorepo-layout.md) | `pnpm i && pnpm build && pnpm test` green; Turborepo caching; Biome; TS strict; dependency rules enforced |
| NB-M0-02 | Local infra via docker compose (Postgres+PostGIS+pgvector, Valkey, MinIO, Mailpit) + Temporal dev | `pnpm dev` boots API, workers, app; health checks pass |
| NB-M0-03 | `packages/contracts` with Zod → OpenAPI 3.1 → `packages/api-client` | CI fails on breaking OpenAPI changes without a version bump |
| NB-M0-04 | Auth: email/phone OTP, Apple, Google; refresh rotation; device-bound refresh in Tauri keychain | E2E sign-in on web and desktop |
| NB-M0-05 | Design tokens + base components in `packages/ui` with Storybook | Light/dark, reduced motion, axe clean |
| NB-M0-06 | Tauri 2 shell wrapping the app; CSP, capabilities; mic & location usage strings; Linux media permission handler | Desktop dev build runs on macOS/Windows/Linux; mic works on all three |
| NB-M0-07 | Observability baseline: OTel traces, Sentry, structured logs with redaction | Trace from client → API → DB visible |
| NB-M0-08 | `ai-kit`: Claude client wrapper (model/effort per route, fallbacks, refusal handling, structured outputs, caching, `ai_runs` telemetry) | Unit tests with recorded fixtures; cost computed per call |

## M1 · People & Context

| ID | Story | AC |
|---|---|---|
| NB-M1-01 | Households & avatars CRUD + illustrated avatar builder | Create/edit/delete; minors flagged; no photo upload for minors |
| NB-M1-02 | Hard constraints with explicit confirmation | Changing constraints requires confirm step; audit log entry |
| NB-M1-03 | Health & culture opt-in with consent ledger; encryption of 🔒 fields | Consent recorded; withdrawal purges data within 1 min |
| NB-M1-04 | Places: GPS/IP locate → reverse geocode → confirm → save; service-area check | GPS ≠ address flow per [WF/05](../02-workflows/05-location-gps-and-addresses.md) |
| NB-M1-05 | Situation Brief builder (time, slot, place, weather, table, occasion detection from text) | Deterministic tests for 50 contexts |
| NB-M1-06 | Dial engine (`packages/domain/dials.ts`) with signals, priors, overrides, hysteresis, reasons | Property tests; 200-situation fixture ≥ 90 % lane agreement |
| NB-M1-07 | Ruleset & knowledge loaders with JSON-Schema validation; CI check | Invalid YAML fails CI; safety rules non-disableable |
| NB-M1-08 | Onboarding flow incl. taste game (10 swipes) | ≤ 2 min median in usability test |

## M2 · Catalog & Ingestion

| ID | Story | AC |
|---|---|---|
| NB-M2-01 | Merchant model, locations (PostGIS), hours, channels with opt-in evidence | Open-now queries correct across time zones |
| NB-M2-02 | Partner console v0: onboarding, channels, opt-in, menu upload | Merchant onboards in ≤ 15 min |
| NB-M2-03 | MenuIngestionWorkflow: extract (vision) → validate → normalize → review → publish | Golden menus F1 ≥ 0.95 names/prices |
| NB-M2-04 | Canonical dishes seed (≥ 600 dishes across launch-zone cuisines) + mapping | ≥ 90 % items mapped with confidence ≥ 0.7 |
| NB-M2-05 | Enrichment (allergen risk, nutrition ranges, flavor, thermal class, travel score, tags) via Batch | Estimated fields flagged; never shown as "free of" |
| NB-M2-06 | Hybrid search (FTS + vector + geo + open-now + constraints) | p95 ≤ 150 ms on seeded metro |
| NB-M2-08 | Overture + OSM import per metro bbox into the directory with license provenance (`catalog.merchant_sources`); dedupe by phone/name/distance/domain | Pilot zone directory built; duplicates < 3 % on a 200-record sample |
| NB-M2-09 | `GooglePlacesAdapter` (runtime only; place-ID storage; Google attribution/map rules) | No Google content persisted beyond place IDs (test) |
| NB-M2-10 | Neighborhood Scout workflow (H3 cells, Places live data, Claude web_search/web_fetch with blocked domains, ParsedMenu drafts) | ≥ 50 % of zone restaurants with website get a draft menu; no marketplace/Meta fetches in logs |
| NB-M2-11 | Outreach CRM: prospect queue by demand, call scripts (EN/ES), wa.me JOIN links, opt-in capture | Opt-in evidence stored for every WhatsApp-enabled kitchen |
| NB-M2-12 | `BusinessProfileMenuAdapter` (owner OAuth `getFoodMenus`) | Connected kitchen's Google menu imported and reviewable |
| NB-M2-07 | Knowledge seed: launch-zone cultural packs, occasions calendar (12 months), natural-medicine top-60 ingredients, allergen ontology | Reviewed by advisors (labels on PR) |

## M3 · Conversation & Council

| ID | Story | AC |
|---|---|---|
| NB-M3-01 | Sessions + SSE event stream (`assistant.delta`, `ui.*`, `dials.changed`) | Reconnect resumes stream; ordered events |
| NB-M3-02 | Host loop with tools (get_situation, set_table, ask_user, consult_council, search_food_graph, compose_meal_sets, quote_logistics, show_proposals, propose_memory, set_dials, handoff_to_human) | Journey evals pass; no forced tool_choice; tool inputs schema-validated |
| NB-M3-03 | Faculties F0–F12 as prompt files + AdviceCard outputs; parallel runner with deadlines | Safety required; others droppable; faculty evals pass |
| NB-M3-04 | Synthesizer (priority merge) in `packages/domain` | Unit tests incl. conflicts (halal vs dish, allergy vs taste) |
| NB-M3-05 | Meal-set composer (beam search + rerank) + deterministic validator | 150 group scenarios: 0 constraint violations |
| NB-M3-06 | Proposal UI (express cards, guided carousel, scenic stories) + Table view (drag + accessible move) | Usability: users assemble a 4-seat set in ≤ 60 s |
| NB-M3-07 | Why-this sheet with reasons, citations, evidence levels, memories used | Every claim links to a kn code or menu field |
| NB-M3-08 | Question selection by expected information gain + question bank | ≤ 2 questions median in guided lane |

## M4 · Liaison & Ordering

| ID | Story | AC |
|---|---|---|
| NB-M4-01 | Basket per leg, price breakdown (estimate) | Totals match ledger rules |
| NB-M4-02 | `MerchantChannel` adapters: WhatsApp (Cloud API) + SMS (Twilio) + partner console | Templates approved; webhooks verified; opt-out honored |
| NB-M4-03 | AvailabilityCheckWorkflow with escalation ladder, holds, backups | Restaurant simulator: p90 ≤ 5 min; backups offered on timeout |
| NB-M4-04 | Liaison reply parser (buttons + free text + voice notes) | ≥ 97 % on parsing eval |
| NB-M4-05 | Checkout: PaymentIntent manual capture, Apple/Google Pay, 3DS | Auth only after all legs available |
| NB-M4-06 | OrderSaga: firm confirm (allergy line), merchant payment (Connect or single-use Issuing card), compensations | Chaos tests: merchant reject, card decline, timeout |
| NB-M4-07 | `DeliveryProvider` adapters: Nash and/or DoorDash Drive, Uber Direct (sandbox) — single mode | Quote/create/cancel/webhooks; failover tested |
| NB-M4-08 | Tracking (WS) + notifications (push/web push/email) | Status latency ≤ 5 s from webhook |
| NB-M4-09 | Capture, ledger, reconciliation job | Daily reconciliation report zero-diff on test data |
| NB-M4-10 | Feedback per seat → preferences; Memory Curator with confirmations | "What I know" reflects new memories with provenance |

## M4b · Cluster Runs (pilot)

| ID | Story | AC |
|---|---|---|
| NB-M4b-01 | Kitchen-cluster rule in composer (metro config: max pair distance, extra minutes, max kitchens, customer distance; co-located = one stop) | 0 multi-kitchen sets violate the rule in 150 scenarios |
| NB-M4b-02 | `own_fleet` DeliveryProvider via Onfleet linked tasks (pickups in order + dropoff, same driver); webhooks → `DeliveryEvent` incl. `purchase_made` | Simulator run with 3 pickups completes; events ordered |
| NB-M4b-03 | Route sequencing (thermal order, ready_by windows) in `packages/domain/routing.ts` | Hot food last; first-hot-pickup→door ≤ 25 min in fixtures |
| NB-M4b-04 | Run cards: Stripe Issuing card per run, limits (leg × 1.10, card-present, allowed categories), real-time authorization matching open stops | Test-mode authorizations: wrong merchant/amount declined; correct approved |
| NB-M4b-05 | Receipt capture & reconciliation (photo, OCR total vs authorization, mismatch queue) | Mismatch > 10 % lands in ops queue |
| NB-M4b-06 | Mode selection & cost model (cluster run vs DaaS legs; crew availability from dispatch tool) | Chooses cheaper feasible mode in fixture matrix |
| NB-M4b-07 | `courier_paid_pickup_v1` template + liaison flow | Kitchen receives exact amount and courier name; ack parsed |
| NB-M4b-08 | Ops run board (live runs, stops, card spend, receipts, reassign) | Ops can reassign a stop to DaaS fallback in ≤ 3 clicks |

## M5 · Voice & Polish

| ID | Story | AC |
|---|---|---|
| NB-M5-01 | Voice gateway (WS): STT/TTS adapters, VAD, barge-in, keyterms | p50 ≤ 1.2 s end-of-speech → first audio on desktop |
| NB-M5-02 | Host orb (Rive) state machine + captions | All states covered; reduced-motion fallback |
| NB-M5-03 | Express lane end-to-end (work lunch journey) | p50 time-to-order ≤ 90 s |
| NB-M5-04 | Spanish localization (UI + AI + templates) | Multilingual eval ≥ 95 % |
| NB-M5-05 | Accessibility pass (WCAG 2.2 AA) | axe clean; manual SR pass |

## M6 · Beta Hardening

| ID | Story | AC |
|---|---|---|
| NB-M6-01 | Full eval suite in CI nightly + gates | Safety 100 %; journey regression ≤ 2 pts |
| NB-M6-02 | Load test (k6): 200 concurrent sessions, 50 concurrent orders | SLOs met |
| NB-M6-03 | Security review + pen test fixes | No high findings open |
| NB-M6-04 | Ops console v0 (live orders, threads, overrides, refunds within limits) | Runbooks executable from console |
| NB-M6-05 | Privacy: DSAR tooling, consumer-health-data policy, consent UX review | Counsel sign-off |
| NB-M6-06 | Closed beta onboarding (≈ 200 households, 30–60 merchants) | G1 metrics tracked on dashboard |

---

## Phase 2+ epics (titles)

- E-201 Cluster runs at metro scale (zones, shifts, crew analytics) · E-202 Shop extras & substitutions · E-203 Synchronized DaaS legs & merged tracking · E-204 Co-located (food hall) pickups · E-205 POS integrations · E-206 Mobile builds (iOS/Android) · E-207 Hands-free voice on LiveKit · E-208 Group links · E-209 AI voice-call escalation · E-210 Natural-medicine content GA · E-211 Public launch ops
- E-301 Canada (FR-CA, data residency) · E-302 Crew model per city · E-303 Calendar integrations · E-304 Rituals & reminders · E-305 Membership
- E-401 Uber Consumer Delivery API · E-402 LATAM (DR) adapters · E-403 B2B · E-404 Licensed alcohol partners
