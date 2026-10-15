# 05 · API & Events

> Status: Draft v1 · REST + OpenAPI 3.1 generated from Zod (`packages/contracts`) · Streaming via SSE (text) and WebSocket (voice, tracking).

## 1. Conventions

- Base: `https://api.<domain>/v1`; partner API under `/partner/v1`; webhooks under `/hooks/*`.
- Auth: OAuth 2.1 / OIDC tokens (access 15 min, refresh rotated); device-bound refresh tokens in the Tauri keychain.
- Idempotency: `Idempotency-Key` header required on POSTs creating baskets, checks, payments, orders.
- Errors: RFC 9457 problem+json `{type, title, status, detail, code, traceId}`.
- Pagination: cursor (`?cursor=…&limit=…`).
- Locale: `Accept-Language`; currency follows the delivery place.

## 2. Eater API

| Method & path | Purpose |
|---|---|
| `POST /auth/otp/start`, `POST /auth/otp/verify`, `POST /auth/oauth/{apple|google}` | Sign-in |
| `GET /me`, `PATCH /me` | Profile, locale, defaults |
| `GET/POST /households`, `GET/PATCH/DELETE /households/{id}` | Households |
| `GET/POST /avatars`, `PATCH/DELETE /avatars/{id}` | Avatars |
| `PUT /avatars/{id}/constraints` | Replace hard constraints (explicit confirmation required) |
| `PUT /avatars/{id}/health` | Opt-in health context (requires consent id) |
| `PUT /avatars/{id}/culture` | Self-declared heritage/observances |
| `GET /memory?avatarId=` , `PATCH /memory/{id}`, `DELETE /memory/{id}` | "What I know about you" |
| `POST /memory/confirmations/{proposalId}` | Accept/reject a proposed memory write |
| `GET/POST /places`, `POST /places/resolve` | Saved places; resolve GPS → address candidates |
| `POST /sessions` | Start session `{placeId?, householdId?, channel, signals}` → `{sessionId, brief, dials}` |
| `POST /sessions/{id}/messages` | Send text / chip / card action → **SSE stream** of `SessionEvent`s |
| `PATCH /sessions/{id}/dials` | User override ("Just get me food", "Inspire me") |
| `PATCH /sessions/{id}/table` | Set present avatars (seats) |
| `GET /sessions/{id}/voice` (WS upgrade) | Voice stream (PCM16/Opus in, events + TTS audio out) |
| `POST /baskets` | Create from a proposal set or manually |
| `PATCH /baskets/{id}` | Edit items/seats/extras caps |
| `POST /baskets/{id}/availability-checks` | Start merchant checks → `{checkId}`; progress via SSE |
| `POST /baskets/{id}/quote` | Fees, tax, dispatch options (single / synchronized / concierge) |
| `POST /checkout` | `{basketId, dispatchOption, tip, extrasCaps, paymentMethod}` → PaymentIntent client secret |
| `GET /orders`, `GET /orders/{id}` | Orders |
| `GET /orders/{id}/track` (WS) | Live status, courier position, per-leg updates |
| `POST /orders/{id}/cancel` | Cancel (policy-dependent) |
| `POST /orders/{id}/issues` | Missing/wrong item, late, damaged |
| `POST /orders/{id}/feedback` | Per-seat dish ratings + notes |
| `POST /groups` / `POST /groups/{id}/join` | Group ordering invite links |

### `SessionEvent` (SSE `event:` names)

| Event | Payload |
|---|---|
| `assistant.delta` | `{text}` streamed tokens |
| `assistant.done` | `{messageId}` |
| `ui.chips` | `{chips:[{id,label,action}] , multi:boolean}` |
| `ui.proposals` | `{proposalId, sets: MealSet[]}` |
| `ui.table` | `{seats:[{avatarId, itemIds}]}` |
| `dials.changed` | `{urgency, inspiration, directness, scenic, lane, reasons[]}` |
| `brief.updated` | `{brief}` |
| `memory.proposed` | `{proposalId, statements[]}` (needs confirmation for hard facts) |
| `availability.progress` | `{legId, merchantName, channel, status, eta?}` |
| `safety.notice` | `{level, message, resources?}` |
| `error` | problem+json |

## 3. Partner API (restaurants & shops)

| Method & path | Purpose |
|---|---|
| `POST /partner/v1/merchants` | Self-onboarding |
| `PUT /partner/v1/merchants/{id}/channels` | WhatsApp/SMS/voice/POS channels + opt-in capture |
| `POST /partner/v1/menus/imports` | Upload PDF/photo/URL for ingestion |
| `GET/PATCH /partner/v1/menus/{id}` | Review AI-parsed menu, fix items, declare allergens/labels |
| `PATCH /partner/v1/items/{id}/availability` | 86 an item |
| `GET /partner/v1/orders?status=` | Incoming legs |
| `POST /partner/v1/legs/{id}/{accept|reject|ready}` | Partner-app path for confirmations |
| `GET /partner/v1/payouts` | Settlements |

## 4. Webhooks (inbound)

| Path | Source | Handling |
|---|---|---|
| `/hooks/stripe` | Payments, Issuing (`issuing_authorization.request` real-time), Connect | Signature check; Issuing approvals must answer within the provider's real-time window |
| `/hooks/whatsapp` | Meta Cloud API messages + statuses | Verify `X-Hub-Signature-256`; route to liaison |
| `/hooks/twilio/{sms|voice}` | SMS replies, call events | Signature check |
| `/hooks/nash`, `/hooks/doordash`, `/hooks/uber` | Delivery status, courier updates, refunds | De-dupe by event id; signal DispatchRun |
| `/hooks/pos/{square|toast|clover}` | Order status, item availability | Signal OrderSaga / update catalog |

All webhooks: verify → persist raw event → ack fast (2xx) → process asynchronously via Temporal signal.

## 5. Domain events (outbox)

| Event | Emitted when | Consumers |
|---|---|---|
| `session.started` / `session.ended` | Session lifecycle | analytics, memory consolidation |
| `proposal.presented` / `proposal.chosen` | Proposals | learning, evals |
| `basket.availability.completed` | All legs answered | order |
| `order.authorized` | Payment authorized | OrderSaga |
| `order.leg.confirmed` / `order.leg.ready` / `order.leg.picked_up` | Leg progress | tracking, notifications |
| `order.delivered` / `order.cancelled` / `order.refunded` | Terminal states | payments, feedback, memory |
| `dispatch.courier.assigned` / `dispatch.eta.changed` | Courier | tracking |
| `feedback.dish.rated` | Ratings | people (preferences), catalog (dish quality) |
| `menu.version.published` | Ingestion | search index, embeddings |
| `memory.written` / `memory.deleted` | Memory | audit |

## 6. Internal orchestrator tools (contract)

Defined in `packages/contracts/ai-tools.ts` and documented in [AI/01](../03-ai/01-ai-platform-architecture.md#6-orchestrator-tools): `get_situation`, `set_table`, `ask_user`, `consult_council`, `search_food_graph`, `compose_meal_sets`, `quote_logistics`, `show_proposals`, `start_availability_check`, `propose_memory`, `set_dials`, `handoff_to_human`.
