# WF/01 · Order Lifecycle

> Status: Draft v1 · Implemented as Temporal workflows `AvailabilityCheckWorkflow`, `OrderSaga`, `DispatchRun` · State machines in `packages/domain/order-state.ts`

## 1. End-to-end flow

```text
 ① Conversation & context      Session → Situation Brief → dials → council
 ② Proposals                   Meal sets (per seat, ≤ 3 merchants) + logistics quote
 ③ Basket                      User accepts/edits; adds shop extras with caps
 ④ Check & hold                Availability checks to all merchants in parallel (soft hold)
 ⑤ Pay                         Customer authorizes total (+ extras cap) via Stripe
 ⑥ Firm confirm & pay merchants Each leg confirmed; merchant paid (Connect / virtual card)
 ⑦ Dispatch                    Single / synchronized legs / concierge run
 ⑧ Pickups & extras            Per-stop status; courier buys extras with capped card
 ⑨ Delivery                    Proof of delivery; capture actual amount
 ⑩ Feedback & memory           Per-seat ratings → preferences, merchant reliability
```

## 2. Order states

```text
DRAFT ─▶ BASKET_READY ─▶ CHECKING ─┬─▶ AVAILABLE ─▶ AUTHORIZING ─▶ AUTHORIZED ─▶ CONFIRMING ─▶ CONFIRMED
                                   │                      │                                     │
                                   └─▶ NEEDS_CHANGES ─▶ (user edits) ─▶ CHECKING                │
                                                          └─▶ PAYMENT_FAILED                     ▼
                                                                                         DISPATCHING ─▶ IN_PROGRESS
                                                                                                          │
                                                                    DELIVERED ◀── ARRIVING ◀── EN_ROUTE ◀─┘
                                                                        │
                                                                     SETTLED ─▶ CLOSED
Exceptional: CANCELLED (any time before CONFIRMED free; after: policy), PARTIALLY_DELIVERED, REFUNDED, FAILED
```

### Leg states (per merchant)

```text
PENDING_CHECK → AVAILABLE | MODIFIED | UNAVAILABLE | NO_ANSWER
AVAILABLE → CONFIRMED → PAID → PREPARING → READY → PICKED_UP
any → CANCELLED (with compensation)
```

### Transition guards (examples)

| Transition | Guard |
|---|---|
| `CHECKING → AVAILABLE` | All legs `AVAILABLE` (or user accepted `MODIFIED` changes) and holds not expired |
| `AVAILABLE → AUTHORIZING` | User tapped Pay; deterministic validator OK (constraints, prices, hours, address, totals) |
| `AUTHORIZED → CONFIRMING` | Payment authorization succeeded; hold still valid (else re-check legs that expired) |
| `CONFIRMED → DISPATCHING` | All legs `CONFIRMED`+`PAID`; dispatch time = max(ready_by) − travel offsets |
| `DELIVERED → SETTLED` | Capture succeeded (≤ authorized); merchant payouts scheduled; courier costs booked |

## 3. AvailabilityCheckWorkflow

```text
input: basketId, legs[], requestedReadyBy per leg, holdMinutes=10
parallel for each leg:
   if merchant has POS adapter → query live availability (≤ 5 s) → result
   else → liaison.sendCheck(leg, channel=primary)
          wait for signal `reply` or timer T1 (soft, e.g., 2 min) → escalate to next channel
          wait T2 (e.g., 4 min) → escalate (AI voice call) → T3 (6 min) → ops queue
          parse reply → AVAILABLE / MODIFIED(substitutions, new ready time, price change) / UNAVAILABLE
emit progress events to the session (SSE) after every state change
when all legs final OR user cancels OR global timeout (8 min):
   if any UNAVAILABLE/MODIFIED → Host proposes alternatives (pre-computed backup per leg)
   return results; start hold timers
```

Backups: when proposals are composed, the Logistics Planner pre-computes a **backup item/merchant per leg** so the UI can offer a one-tap swap without restarting the conversation.

## 4. OrderSaga (after authorization)

```text
1. confirmLegs: for each leg in parallel → liaison.sendFirmConfirmation(leg, readyBy, allergyNotes, payment info)
     wait for ack (button/POS) with timeout; on reject → compensate (see §6)
2. payMerchants: per leg payment method
     connect_settlement → ledger entry (merchant payable), transfer on payout schedule
     virtual_card       → create single-use Issuing card (amount × 1.05 cap, food MCCs, 6 h expiry) → share via secure merchant flow / ops call
     courier_card       → (concierge run) amount added to run card limit
3. startDispatch: DispatchRun(child workflow) with mode & schedule
4. track: signals from dispatch & merchants → update legs → notify user
5. complete: delivered → capture (actual items + actual extras ≤ cap + tip) → close leg payments → feedback prompt
```

## 5. Timing model (kitchen sync)

For each leg: `ready_by = pickup_eta(leg) − buffer(thermal_class)`.

| Thermal class | Buffer | Pickup order (concierge run) |
|---|---|---|
| Ambient (drinks, bread, flowers) | ±10 min | First |
| Cold (salads, sushi, desserts) | 5 min | Early |
| Frozen (ice cream) | 0–3 min | Last-but-one or insulated bag |
| Hot (soups, fried, grills) | 0–5 min | Last |

Synchronized legs (DaaS): choose per-leg dispatch times so that predicted arrivals fall inside the promise window (default 10 min). If spread > window, Logistics proposes a different merchant combination or scheduled time.

## 6. Compensations

| Failure | Compensation |
|---|---|
| A merchant rejects at firm confirm | Offer backup (re-check) → if user declines, cancel that leg; reduce authorization (partial reversal) or capture less later |
| Payment authorization fails | Keep holds briefly, let user retry/change method; release holds on timeout |
| Virtual card declined at merchant | Ops override path; new card; or merchant settles later via Connect |
| Courier no-show/cancel | Dispatch failover; if ETA slips beyond window → notify user, offer credit |
| Leg picked up incomplete | Partial refund for missing items automatically; merchant reconciliation |
| User cancels after confirmation | Policy: before cooking start (ready_by − prep_time) → full refund minus small fee; after → refund delivery only, food per merchant policy |

## 7. Notifications (customer)

| Moment | Channel |
|---|---|
| Checking kitchens (live chips) | In-app SSE |
| All confirmed + ETA | In-app + push |
| Courier assigned / picked up leg X | In-app + push (collapsed) |
| Arriving in ~5 min | Push (+ SMS if opted in) |
| Delivered + photo proof | Push |
| Issue resolved / refund | Push + email receipt |

Order tracking map: one courier path with stop icons (🍜 → 🌮 → 🛒 → 🏠) for concierge runs; for synchronized legs, multiple couriers converging with "arriving within 8 min of each other".
