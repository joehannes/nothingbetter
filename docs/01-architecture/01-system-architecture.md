# 01 · System Architecture

> Status: Draft v1 · Owner: Engineering · Decisions recorded in [ADRs](adr/)

## 1. Context (C4 level 1)

```text
                          ┌──────────────────────────────┐
   Eater (voice/text) ───▶│  NothingBetter clients        │
                          │  Tauri desktop+mobile, Web    │
                          └──────────────┬───────────────┘
                                         │ HTTPS (REST + SSE/WebSocket), WebRTC/WS audio
                          ┌──────────────▼───────────────┐
 Restaurant/shop staff ──▶│  NothingBetter platform       │◀── Ops & support staff
 (WhatsApp/SMS/voice/POS) │  (API, AI, workflows, data)   │
                          └──┬────────┬────────┬────────┬─┘
                             │        │        │        │
                ┌────────────▼┐ ┌─────▼─────┐ ┌▼──────┐ ┌▼──────────────────┐
                │ Anthropic   │ │ Stripe    │ │ Meta  │ │ Nash → DoorDash   │
                │ Claude API  │ │ Payments, │ │ WA    │ │ Drive, Uber Direct│
                │             │ │ Issuing,  │ │ Cloud │ │ + own fleet       │
                │ Voice STT/  │ │ Connect,  │ │ API,  │ └───────────────────┘
                │ TTS vendors │ │ Tax       │ │ Twilio│  Maps/geo (Mapbox),
                └─────────────┘ └───────────┘ └───────┘  POS (Square/Toast/Clover)
```

## 2. Containers (C4 level 2)

```text
┌────────────────────────────────────── Clients ──────────────────────────────────────┐
│ apps/app (React SPA) ── packaged by ── apps/tauri (desktop + iOS/Android shell, Rust)│
│ apps/site (Astro marketing) · apps/partner (restaurant console) · apps/ops (internal) │
└───────────────┬──────────────────────────────────────────────────────────────────────┘
                │
┌───────────────▼─────────────────┐   ┌──────────────────────────────┐
│ api-gateway (Fastify, Node 24)   │──▶│ realtime (WS/SSE fan-out,     │
│ auth, REST, rate limits, OpenAPI │   │ Redis pub/sub)                │
└───┬─────────┬─────────┬─────────┘   └──────────────────────────────┘
    │         │         │
┌───▼───┐ ┌───▼──────┐ ┌▼──────────────┐ ┌───────────────────┐ ┌──────────────────┐
│ conv- │ │ catalog  │ │ order-service │ │ liaison-service   │ │ voice-gateway    │
│ ersa- │ │ (food    │ │ (baskets,     │ │ (WhatsApp, SMS,   │ │ (STT/TTS stream, │
│ tion  │ │ graph,   │ │ orders, pay-  │ │ voice, POS        │ │ VAD, barge-in)   │
│ (AI   │ │ search)  │ │ ments, dis-   │ │ adapters)         │ └──────────────────┘
│ orch.)│ └───┬──────┘ │ patch)        │ └─────────┬─────────┘
└───┬───┘     │        └──────┬────────┘           │
    │         │               │                    │
┌───▼─────────▼───────────────▼────────────────────▼─────────────┐
│ Temporal (durable workflows): OrderSaga, AvailabilityCheck,      │
│ DispatchRun, MenuIngestion, Enrichment, MemoryConsolidation       │
└───┬───────────────────────────────────────────────────────────────┘
    │
┌───▼──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
│ PostgreSQL 17     │ │ Redis/Valkey │ │ Object store │ │ Knowledge repo    │
│ + PostGIS +       │ │ cache, rate, │ │ (S3/R2):     │ │ (git: rulesets,   │
│ pgvector          │ │ pub/sub      │ │ menus, media │ │ cultures, templ.) │
└───────────────────┘ └──────────────┘ └──────────────┘ └──────────────────┘
```

**Deployment shape:** start as a **modular monolith** (`services/platform`) with these modules as separate packages and one deployable, plus separate deployables for `voice-gateway` and the Temporal workers. Split modules into services only when scale or team boundaries demand it ([ADR-0003](adr/ADR-0003-modular-monolith-temporal.md)).

## 3. Module responsibilities

| Module | Owns | Key dependencies |
|---|---|---|
| `identity` | Users, sessions, devices, OAuth (Apple, Google, email OTP), consents | — |
| `people` | Households, avatars, hard constraints, preferences, memory | identity |
| `conversation` | Sessions, messages, Situation Brief, dials, orchestrator, faculties | people, catalog, knowledge, ai |
| `ai` | Claude client, prompt/ruleset registry, structured outputs, telemetry, evals hooks | knowledge |
| `knowledge` | Cultural, health, natural-medicine, occasion, season knowledge; loaders from `knowledge/` | — |
| `catalog` | Restaurants, shops, menus, canonical dishes, enrichment, search (FTS + vector + geo) | knowledge, ai |
| `geo` | Addresses, geocoding, service areas, routes, ETAs, detour search | Mapbox |
| `order` | Baskets, meal sets, orders, legs, state machine, pricing, fees, tax | catalog, payments, liaison, dispatch |
| `liaison` | Merchant channels (WhatsApp, SMS, voice, POS, partner app), templates, reply parsing | ai |
| `payments` | PaymentIntents, Issuing cards, Connect transfers, refunds, ledger | Stripe |
| `dispatch` | Quotes, provider selection, cluster runs, synchronized legs, tracking | Nash, DaaS, geo |
| `notifications` | Push (APNs/FCM/web push), email, in-app | — |
| `ops` | Support tooling, overrides, incident flags, audit | all |

Module rule: **modules talk through their public TypeScript interfaces and domain events**, never each other's tables.

## 4. Key runtime flows

### 4.1 Conversation turn (text)

```text
client ──POST /v1/sessions/:id/messages──▶ api ──▶ conversation.handleTurn()
  1. load Session + Situation Brief (cached) + present avatars (people)
  2. dial engine updates dials from signals            (deterministic, < 5 ms)
  3. ruleset engine → active faculties, constraints    (deterministic)
  4. orchestrator (Claude, streaming) decides: ask / consult / propose
       └─ tool: consult_council → faculties in parallel (Claude, structured outputs)
       └─ tool: search_food_graph → catalog (SQL + vector + geo)
       └─ tool: compose_meal_sets → deterministic optimizer + LLM rerank
       └─ tool: quote_logistics → dispatch quotes (cached 2 min)
  5. stream tokens + UI directives (chips, cards) over SSE
  6. memory curator proposes memory writes (async, needs confirmation for hard facts)
```

### 4.2 Order (multi-restaurant)

```text
basket accepted → AvailabilityCheck workflow (fan-out per leg, timers, escalation)
  → all legs available → user pays (PaymentIntent authorize)
  → OrderSaga: firm-confirm legs → pay merchants (Connect / single-use virtual card)
  → DispatchRun: cluster run, single/co-located DaaS leg, or synchronized legs → tracking events
  → delivered → capture → payouts → feedback prompt
compensations: void auth, cancel legs, refund, re-dispatch
```

Details: [WF/01 Order lifecycle](../02-workflows/01-order-lifecycle.md).

## 5. Cross-cutting concerns

| Concern | Approach |
|---|---|
| **Consistency** | Postgres transactions inside a module; Temporal sagas across modules; transactional outbox for events |
| **Idempotency** | Idempotency keys on all POSTs that create money/order objects; provider webhooks de-duplicated by event id |
| **Time** | Store UTC (`timestamptz`); every place and restaurant carries an IANA time zone; "ready by" times computed in merchant TZ |
| **Money** | Integer minor units + ISO 4217 currency; never floats; double-entry ledger table for every money movement |
| **Observability** | OpenTelemetry traces across API → Temporal → providers → Claude; Sentry errors; per-session AI cost |
| **Feature flags** | Per user/household/metro flags (e.g., `cluster_runs`, `natural_medicine_lens`, `voice_handsfree`) |
| **Config by metro** | Service areas, fees, tax rules, enabled providers, languages, cultural packs |
| **Security & privacy** | See [06 Security, privacy & compliance](06-security-privacy-compliance.md) |

## 6. Non-functional targets (v1)

| Target | Value |
|---|---|
| First token of assistant reply (text) | p50 ≤ 1.2 s, p95 ≤ 2.5 s |
| First proposals rendered | p50 ≤ 5 s, p95 ≤ 9 s |
| Voice: end of user speech → first audio | p50 ≤ 1.2 s, p95 ≤ 2.0 s |
| API availability (ordering path) | 99.9 % monthly |
| Order state durability | No order state lost on process crash (Temporal) |
| AI cost per completed order | Tracked; budget set after Phase 1 baseline |
| Data residency | US users in US region; Canadian users' data in Canada region (Quebec Law 25 readiness) |

## 7. Evolution path

1. **Phase 1:** modular monolith + Temporal workers + voice gateway; one region.
2. **Phase 2:** split `catalog` ingestion workers and `liaison` (high webhook volume) as separate deployables.
3. **Phase 3:** second region (Canada), scaled own-fleet dispatch, read replicas, dedicated search (Typesense/OpenSearch) if FTS+pgvector p95 > 150 ms.
