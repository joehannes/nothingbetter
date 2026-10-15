# Plan/03 · Testing Strategy

> Status: Draft v1

## 1. Pyramid

| Layer | Tooling | Focus |
|---|---|---|
| Unit | Vitest | `packages/domain` (dials, synthesizer, composer, pricing, state machines, routing), parsers, validators |
| Property | fast-check | Hard-constraint invariants ("no item violating a seat's constraint ever passes the validator"), pricing totals, ledger balance |
| Integration | Vitest + Testcontainers (Postgres, Valkey) + Temporal test env | Module APIs, workflows with time skipping, webhooks |
| Contract | Provider fakes + recorded sandbox fixtures | Stripe, Nash/DoorDash/Uber, WhatsApp, Twilio, Mapbox, STT/TTS |
| E2E | Playwright (web), Tauri WebDriver smoke (desktop) | Journeys A–G from [personas](../00-product/03-personas-and-journeys.md) with simulators |
| AI evals | `evals/` harness | See [AI/11](../03-ai/11-evaluation-and-quality.md) |
| Load | k6 | Sessions, SSE/WS fan-out, webhook bursts |
| Chaos | Fault injection in provider fakes | Timeouts, duplicates, out-of-order webhooks, declines |
| Accessibility | axe (Storybook + Playwright), manual SR | WCAG 2.2 AA |
| Security | SAST, dependency audit, secret scanning, pen test | OWASP ASVS L2 targets |

## 2. Simulators (in `packages/testing`)

- **Restaurant simulator:** merchant personas (fast/slow/rush-hour/Spanish/voice-note/partial availability/price change/angry) answering WhatsApp/SMS via fake channel adapters.
- **Courier/dispatch simulator:** quotes, assignment delays, cancellations, ETA drift, GPS traces.
- **Payment simulator:** Stripe test clocks/cards for declines, 3DS, partial capture; Issuing test authorizations.
- **User simulator:** persona-driven multi-turn conversations for AI journey evals.

## 3. Critical invariants (must have property tests)

1. A proposal/basket never contains an item conflicting with any present seat's hard constraint.
2. Captured amount ≤ authorized amount; ledger entries balance per transaction.
3. No availability check is sent without an explicit user "Check & hold" action.
4. No customer authorization happens before all legs are `AVAILABLE` (unless `auth_then_check` flag).
5. No WhatsApp/SMS message to a merchant without opt-in evidence; none during quiet hours.
6. Health/culture data never appears in analytics events or logs.
7. A Safety Sentinel failure blocks proposals.

## 4. Environments & data

Synthetic seeded metro (`seed:miami-demo`) with 60 merchants, 4,000 menu items, 30 households; anonymized beta data only in staging with approval.
