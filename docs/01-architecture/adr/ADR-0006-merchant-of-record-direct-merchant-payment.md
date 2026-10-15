# ADR-0006 · Merchant of record; kitchens paid by us or by our courier's capped card

- Status: Accepted (2026-10-15) · **Revised 2026-10-15 (rev 2)** — courier-paid pickups move into the Phase 1 cluster-run pilot.

## Context
The founder's model: customer pays NothingBetter; the courier pays each restaurant at pickup and buys extras on the way. DaaS couriers (DoorDash Drive, Uber Direct) pick up prepaid orders; DoorDash's Red Card exists only in its Shop & Deliver retail integration.

## Decision
- NothingBetter sells the whole order (MoR) and charges the customer via Stripe (authorize at checkout, capture after delivery).
- Kitchens are paid by one of:
  1. **Courier run card** (cluster runs): Stripe Issuing card per run; spending limit per authorization = planned leg amount × 1.10; allowed categories restaurants/fast food/grocery/florists; card-present only; expires at run end; real-time authorization checks the run's open legs; receipt photo per purchase. **This is the founder's model, without cash.**
  2. **Connect settlement** for partner restaurants on prepaid legs (DaaS or cluster runs).
  3. **Single-use virtual card** (card-not-present) for non-partner restaurants on DaaS legs, provided at firm confirmation.
  4. **DoorDash `red_card`** if DoorDash enables it for our account (DoorDash invoices us for items).
- DaaS fees are paid to the provider per delivery; crew pay via contract invoice or Connect payouts.

## Consequences
- + No cash; every purchase is controlled and reconcilable.
- − Card-present spending by couriers needs fraud controls and receipt audit; Stripe Issuing program approval is on the Phase 1 critical path.
