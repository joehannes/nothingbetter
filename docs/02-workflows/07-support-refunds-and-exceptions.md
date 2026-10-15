# WF/07 · Support, Refunds & Exceptions

> Status: Draft v1

## 1. Self-serve first

In-app "Something wrong?" on the order page:

| Issue | Flow | Default resolution |
|---|---|---|
| Missing item | Select items → optional photo | Instant refund/credit within limits |
| Wrong item | Select item → photo | Refund item; redelivery if feasible and wanted |
| Cold/damaged | Select → photo | Credit (partial) |
| Late | Automatic detection | Delivery-fee credit if > 20 min late |
| Allergy concern | Dedicated button | Immediate human escalation; full refund; incident |
| Courier issue | Free text | Ops review; provider escalation |

## 2. Support agent (AI) + humans

- AI support agent (Claude, tools limited to: read order, issue credits within policy, create ticket, message merchant via liaison, schedule callback). No refunds above policy without a human.
- Human escalation always available ("Talk to a person"); SLA during service hours: first response ≤ 2 min in-app chat.
- Merchant-side issues handled via liaison threads; repeated issues lower reliability scores.

## 3. Exception catalog (ops runbook excerpts)

| Exception | Detection | Action |
|---|---|---|
| Kitchen silent after firm confirm | No ack in 4 min | Ops call; backup kitchen; user notified |
| Courier waiting > 10 min at kitchen | Provider event | Ready ping; consider releasing courier & re-dispatch |
| Virtual card declined | Issuing event | Check controls; issue new card; or Connect settlement |
| Price mismatch at pickup (> tolerance) | Courier/merchant report | Approve within tolerance; otherwise ask user |
| Address unreachable | Courier report | Call user; hold up to 5 min; return policy |
| Severe weather | Weather alert | Pause concierge runs; widen windows; inform users |

## 4. Learning loop

All issues tagged (merchant, provider, item, cause) → weekly review → menu fixes, merchant coaching, ruleset tweaks (e.g., travel score of fries), provider strategy changes.
