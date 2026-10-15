# WF/03 · Dispatch, Multi-Pickup & Shop Extras

> Status: Draft v1 · Decision: [ADR-0005](../01-architecture/adr/ADR-0005-delivery-orchestration.md) · Provider facts: [research §2](../00-product/02-research-and-partner-landscape.md#2-delivery-as-a-service-daas-options)

## 1. Dispatch modes

| Mode | When | How | Customer sees |
|---|---|---|---|
| **single** | One merchant | One DaaS delivery (via Nash or direct DoorDash Drive / Uber Direct) | One courier |
| **synchronized_legs** | 2–3 merchants, DaaS only (Phase 2) | N deliveries, each scheduled so arrivals fall in one promise window | "2 couriers, arriving within 8 min of each other"; fees per leg |
| **batched_job** | Provider supports multi-pickup single-drop (verify per provider) | One provider job with multiple pickups | One courier |
| **concierge_run** | Launch zone with own/partner pool (Phase 3) | One courier, ordered stops (pickups + shop extras), capped Issuing card | One courier, route with stops |

The Logistics Planner quotes all feasible modes and the Host presents the trade-off ("One courier: $11.90, ~45 min. Two couriers: $17.50, ~38 min.").

## 2. `DeliveryProvider` interface

```ts
interface DeliveryProvider {
  id: "nash" | "doordash_drive" | "uber_direct" | "own_fleet";
  capabilities(): Promise<{ multiPickup: boolean; scheduled: boolean; alcohol: boolean;
                            maxDistanceMi?: number; regions: string[] }>;
  quote(req: QuoteRequest): Promise<Quote[]>;           // fee, pickup/dropoff ETAs, expiry
  create(req: CreateDeliveryRequest): Promise<DeliveryRef>;
  cancel(ref: DeliveryRef, reason: string): Promise<CancelResult>;
  get(ref: DeliveryRef): Promise<DeliveryStatus>;
  // webhooks normalized to DeliveryEvent: assigned, at_pickup, picked_up, en_route, at_dropoff, delivered, failed, cancelled, eta_changed
}

type CreateDeliveryRequest = {
  externalId: string;                       // leg or run id (idempotency)
  pickups: Stop[];                          // 1 for DaaS single; N for batched/concierge
  dropoff: Stop & { instructions?: string; contactless?: boolean; verification?: "photo" | "pin" };
  pickupWindow?: { start: string; end: string };
  dropoffWindow?: { start: string; end: string };
  orderValueMinor: number; currency: "USD" | "CAD";
  items: { name: string; quantity: number; thermalClass?: ThermalClass }[];
  tipMinor?: number;
};
```

## 3. Provider selection (dispatch strategy)

```text
for each feasible mode:
   quotes = parallel provider.quote()  (timeout 3 s, cache 2 min)
   score  = w_fee·fee + w_eta·eta + w_rel·(1−reliability) + w_quality·travelPenalty + w_legs·legs
choose min score subject to: promise window feasible, provider in region, item restrictions (no alcohol v1)
failover: on cancel/no-assign after T (e.g., 6 min) → next provider (Nash strategy or our own loop)
```

Nash adds provider aggregation, contract pricing and auto-failover across its network; direct adapters to DoorDash Drive / Uber Direct are kept as alternates (no single point of failure).

## 4. Synchronized legs (Phase 2)

1. Compute for each leg: `prep_done = ready_by`, `travel = provider pickup→dropoff ETA`.
2. Target arrival window `[T, T+10 min]` (scheduled) or `[now+ETA_max, +10]` (ASAP).
3. Schedule each leg's delivery creation (`pickupWindow`) so predicted arrival ∈ window; kitchens receive matching `ready_by`.
4. Monitor ETAs; if spread grows beyond window, notify user early with honest update.
5. Customer tracking merges legs into one view.

Fees: each leg is a separate delivery fee (e.g., DoorDash Drive's distance-based rate per leg). The Value Analyst and Logistics Planner prefer combinations that reduce legs when the experience is equal.

## 5. Concierge runs (Phase 3)

### Fleet model options

| Option | Pros | Cons |
|---|---|---|
| Partner courier company (local) via Nash/Onfleet | Fast to start, insured, experienced | Margins, fewer controls |
| Own contractor pool (Stripe Connect payouts) | Control, brand experience | Labor-law compliance (min pay rules in some cities, classification), insurance, ops |
| Hybrid | Flexibility | Complexity |

Recommendation: start with a **partner courier company** in the launch zone dispatched through Nash/Onfleet, with our capped Issuing cards for purchases.

### Route planning

```text
stops = restaurant pickups + shop stops
order constraints: ambient/cold first, hot last; shop stops before hot pickups;
                   frozen near end; each pickup after its ready_by − 3 min
objective: minimize total time subject to max route time (e.g., 45 min from first hot pickup to door)
detour budget for extras: ≤ 6 min each, ≤ 12 min total (configurable by dials/occasion)
```

Solved with a small TSP-with-time-windows heuristic (≤ 6 stops) in `packages/domain/routing.ts`, using a distance/time matrix from Mapbox Matrix API.

### Courier purchases

- Courier receives a **run card** (Stripe Issuing virtual card in a wallet or physical card) with:
  - spending limit per authorization = expected leg/extra amount × 1.10,
  - allowed categories: restaurants, fast food, grocery, convenience, florists (no liquor stores in v1),
  - auto-expiry at run end; real-time authorization webhook checks the order context (`issuing_authorization.request`).
- Receipt photo required per purchase in the courier app; mismatch > 10 % → ops review.

## 6. Shop extras

### Discovery along the route

```sql
-- shops within a corridor around the planned route polyline, open at the expected stop time
select s.*
from catalog.merchant_locations s
join catalog.merchants m on m.id = s.merchant_id
where m.kind in ('shop','florist','grocery','bakery')
  and st_dwithin(s.geom, :route_geom, :corridor_m)        -- e.g. 600 m
  and is_open(s.hours, :eta_at_stop, s.timezone);
```

### Catalog for extras

- Partner shops with item catalogs (POS-integrated) → exact items/prices.
- Non-partner shops → **category requests** with caps: "sparkling water, 2 bottles, ≤ $8", "seasonal bouquet ≤ $25", "pint of vanilla ice cream ≤ $9".
- Substitution rule per extra: *same brand any size* · *similar item* · *skip if unavailable* · *message me* (in-app quick approve within 2 min, else rule fallback).

### Restrictions (v1)

No alcohol, tobacco, vapes, cannabis, prescription or pharmacy-only items, gift cards, or items > $50 each without user re-confirmation.

## 7. Tracking

- DaaS: provider tracking URL + webhook statuses → normalized events → WS to clients.
- Concierge runs: courier app location every 10–15 s while active (through dispatch vendor's driver app initially).
- Customer map: route polyline, stop icons, live ETA, per-leg chips ("🌮 picked up").
- Proof of delivery: photo or PIN for gift/high-value orders.

## 8. Edge cases

| Case | Handling |
|---|---|
| Food not ready at courier arrival | Courier waits up to N min (ruleset); notify merchant via liaison "ready?"; re-sequence stops in concierge run |
| One leg's courier fails | Failover; if other legs already arriving, notify user with honest split ETA and credit |
| Shop item unavailable | Substitution rule; refund difference automatically |
| Address issue at dropoff | Courier chat/call via provider; customer instructions; leave at safe place only if allowed |
| Weather/traffic spike | Wider promise windows proactively; fewer legs |
