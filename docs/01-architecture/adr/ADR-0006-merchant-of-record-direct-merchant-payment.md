# ADR-0006 · NothingBetter is merchant of record and pays merchants directly

- Status: Accepted (2026-10-15)

## Context
The original idea was: customer pays us, we pay the delivery service all the money, and the courier pays each restaurant. DaaS couriers don't carry order money; cash/courier-paid flows add fraud and reconciliation risk.

## Decision
- NothingBetter sells the whole order (MoR) and charges the customer via Stripe (authorize at checkout, capture after fulfillment).
- Restaurants are paid by NothingBetter: **Connect settlement** for partner restaurants, or a **single-use Stripe Issuing virtual card** (exact amount + small buffer, food MCCs only, expires in hours) for non-partner restaurants (paid over phone/online ordering at firm confirmation).
- Courier-paid purchases exist only in concierge runs (own/partner pool) with a **capped Issuing card** per run.
- DaaS fees are paid to the provider per delivery.

## Consequences
- + No cash; every purchase is controlled, logged and reconcilable.
- + Matches the user's intent ("courier pays along the way") where it's safe.
- − We carry chargeback and refund exposure; sales-tax collection obligations.
