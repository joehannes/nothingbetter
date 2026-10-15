# ADR-0005 · Delivery: cluster runs for multi-kitchen orders, DaaS for single kitchens

- Status: Accepted (2026-10-15) · **Revised 2026-10-15 (rev 2)** — cluster runs promoted from Phase 3 to a Phase 1 pilot after founder feedback and multi-pickup research.

## Context
Users order from several nearby kitchens and want one delivery, ideally with the courier paying at each counter and buying extras on the way. Research ([§2.2](../../00-product/02-research-and-partner-landscape.md#22-one-trip-many-kitchens--options-checked-2026-10-15)):
- DoorDash Drive and Uber Direct APIs: one pickup → one dropoff per delivery; Nash batch jobs require a single shared pickup.
- DoorDash Drive supports Dasher-paid purchases (`red_card`) only in its Shop & Deliver (retail catalog) integration, enabled via a DoorDash account manager.
- DoorDash DoubleDash and Uber Eats multi-store prove the networks can do multi-pickup, but only in consumer apps.
- Dispatch software (Onfleet linked tasks, Shipday, Nash own-fleet) supports multi-pickup routes for couriers we dispatch.

## Decision
1. `DeliveryProvider` interface with adapters: **own_fleet** (Onfleet or Shipday), **nash**, **doordash_drive**, **uber_direct**.
2. Dispatch modes, chosen per order by the Logistics Planner:
   - `cluster_run` — one courier from our own/partner crew, ordered pickups within the **kitchen-cluster radius** (≈ 1 km, ≤ 8 min extra), courier pays with a capped Stripe Issuing run card, optional shop stops. **Primary multi-kitchen mode; Phase 1 pilot in one zone at peak hours.**
   - `single` — one kitchen, one DaaS delivery (DoorDash Drive / Uber Direct via Nash or direct).
   - `co_located` — food hall / plaza / multi-brand kitchen with a consolidated bag → one DaaS delivery.
   - `synchronized_legs` — fallback when no crew is available: N DaaS deliveries timed to arrive within a 10-minute window.
3. Composer constraint: baskets mixing kitchens must satisfy the cluster rule (config per metro).
4. Business development: ask DoorDash/Uber for multi-pickup and Red Card on Drive/Direct; adopt if granted.

## Consequences
- + Delivers the founder's "one courier, pays along the way" model in Phase 1.
- − Courier operations (crew sourcing, shifts, cards, receipts, insurance, labor compliance) become a Phase 1 workstream ([WF/08](../../02-workflows/08-courier-operations.md)).
- − Economics depend on crew utilization; the dispatcher falls back to DaaS when a run would cost more.
