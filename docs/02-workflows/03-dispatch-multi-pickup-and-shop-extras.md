# WF/03 · Dispatch, Multi-Pickup & Shop Extras

> Status: Draft v2 · Revised 2026-10-15 · Decision: [ADR-0005](../01-architecture/adr/ADR-0005-delivery-orchestration.md) · Provider research: [§2](../00-product/02-research-and-partner-landscape.md#2-delivery) · Crew operations: [WF/08](08-courier-operations.md)

## 1. Dispatch modes

| Mode | When | How | Customer sees |
|---|---|---|---|
| **cluster_run** (primary for 2+ kitchens) | Kitchens satisfy the cluster rule (§2) and a crew courier is available | One courier from our own/partner crew; ordered pickups; pays at each counter with a capped run card; optional shop stops | "1 courier · 3 stops · 7:20–7:30" |
| **single** | One kitchen | One DaaS delivery (DoorDash Drive / Uber Direct, via Nash or direct) | "1 courier" |
| **co_located** | Food hall / plaza / multi-brand kitchen with consolidated bag | One DaaS delivery from the shared address | "1 courier" |
| **synchronized_legs** (fallback) | No crew courier available or outside crew hours | N DaaS deliveries scheduled to land in one 10-minute window | "2 couriers, arriving within 8 min of each other" + per-leg fees |
| **batched_job** (if offered later) | A provider enables multi-pickup on its API (BD ask to DoorDash/Uber) | One provider job with multiple pickups | "1 courier" |

The Logistics Planner quotes all feasible modes and the Host presents the trade-off only when it matters ("One courier: $11.90, ~42 min. Two couriers: $17.50, ~36 min.").

## 2. Kitchen-cluster rule

Applied by the meal-set composer (hard constraint for multi-kitchen sets):

```yaml
# metro config: dispatch.cluster
cluster:
  max_pair_distance_m: 1000        # every kitchen within ~1 km (0.6 mi) of the first pickup
  max_extra_minutes: 8             # extra route time added by additional pickups
  max_kitchens: 3                  # Phase 1 pilot
  max_customer_distance_mi: 5      # keeps DaaS base rate if we fall back to separate legs
  co_located_counts_as_one: true   # food halls, plazas, multi-brand kitchens
shop_extras:
  max_detour_minutes_each: 6
  max_detour_minutes_total: 12
```

Distance surcharges (e.g., DoorDash Drive +$0.75/mi beyond 5 mi up to 15 mi) pay for a longer single leg — they don't buy extra pickups. The cluster rule is what keeps multi-kitchen cost and food quality under control.

## 3. Mode selection (dispatch strategy)

```text
input: meal set (legs with merchant locations, ready_by, thermal classes), customer place, desired window
feasible = []
if legs == 1:
    feasible += single (DaaS quotes)            + cluster_run (if crew idle nearby and cheaper)
if legs >= 2 and cluster_rule_ok(legs):
    feasible += cluster_run (crew availability & ETA from dispatch tool)
    if all legs co-located:      feasible += co_located
    feasible += synchronized_legs (DaaS quotes per leg)
cost(mode)  = courier_cost + card_fees + expected_wait_cost
score(mode) = w_cost·cost + w_eta·eta + w_quality·travel_penalty + w_reliability·(1−reliability)
choose min score; failover order: cluster_run → synchronized_legs → single legs re-timed
```

`courier_cost` for a cluster run = estimated run minutes × crew cost per minute (contract or hourly), + mileage if applicable. Crew utilization is tracked so the estimate reflects reality (a courier idle anyway is nearly free; a courier pulled from another run is not).

## 4. Cluster runs (Phase 1 pilot)

### 4.1 Route planning

```text
stops = restaurant pickups (+ shop stops)
order: ambient/cold first, shop stops before hot pickups, hot food last, frozen near the end (insulated bag)
each pickup ≥ its ready_by − 3 min
objective: minimize time from first hot pickup to door (target ≤ 25 min) subject to max run time (≤ 45 min)
```

Small TSP-with-time-windows heuristic (≤ 6 stops) in `packages/domain/routing.ts` using a Mapbox Matrix. Kitchens receive `ready_by` times matched to the sequence (WF/02).

### 4.2 Dispatch tooling

| Tool | Use |
|---|---|
| **Onfleet** (preferred) | Linked tasks: pickup tasks in order + one dropoff, same driver; driver app with photo proof, barcode, signatures; webhooks |
| Shipday (alternative) | Multi-depot pickups, route planning, driver app; DaaS fallback built in |
| Nash own-fleet routes (alternative) | If Nash is also our DaaS orchestrator |

Adapter: `DeliveryProvider` id `own_fleet` with sub-adapter `onfleet | shipday | nash_fleet`.

### 4.3 Paying at the counter

- Courier carries a **run card** (Stripe Issuing; virtual card in Apple/Google Wallet or a physical card), limits per [WF/04](04-payments-and-money-flow.md#4-paying-merchants).
- Kitchen was told by WhatsApp "Our courier pays at the counter" (template `courier_paid_pickup_v1`) with the exact total.
- Real-time authorization approves only if the merchant matches an open leg of the run (MCC + name/location heuristics) and amount ≤ leg amount × 1.10.
- Receipt photo in the driver app; mismatch > 10 % → ops review.

### 4.4 Pilot scope

- One launch zone (dense restaurant cluster), dinner peak (e.g., 17:00–21:00) + weekend lunch.
- 2–4 couriers per shift; 15–25 partner kitchens inside the zone.
- Outside crew hours: single/synchronized DaaS legs.
- Success = cost per run ≤ equivalent DaaS legs for ≥ 60 % of multi-kitchen orders, on-time ≥ 90 %, receipt mismatches ≤ 2 %.

## 5. `DeliveryProvider` interface

```ts
interface DeliveryProvider {
  id: "own_fleet" | "nash" | "doordash_drive" | "uber_direct";
  capabilities(): Promise<{ multiPickup: boolean; courierPays: boolean; scheduled: boolean;
                            alcohol: boolean; maxDistanceMi?: number; regions: string[] }>;
  quote(req: QuoteRequest): Promise<Quote[]>;           // fee or cost estimate, pickup/dropoff ETAs, expiry
  create(req: CreateDeliveryRequest): Promise<DeliveryRef>;
  cancel(ref: DeliveryRef, reason: string): Promise<CancelResult>;
  get(ref: DeliveryRef): Promise<DeliveryStatus>;
  // webhooks normalized to DeliveryEvent: assigned, at_pickup(stop), picked_up(stop), purchase_made(stop),
  // en_route, at_dropoff, delivered, failed, cancelled, eta_changed
}

type CreateDeliveryRequest = {
  externalId: string;                       // run or leg id (idempotency)
  pickups: (Stop & { readyBy: string; payAtCounter?: { amountMinor: number; currency: "USD" | "CAD" } })[];
  shopStops?: (Stop & { shoppingList: ShoppingItem[]; capMinor: number })[];
  dropoff: Stop & { instructions?: string; contactless?: boolean; verification?: "photo" | "pin" };
  dropoffWindow?: { start: string; end: string };
  orderValueMinor: number; currency: "USD" | "CAD";
  items: { name: string; quantity: number; thermalClass?: ThermalClass; stopRef: string }[];
  tipMinor?: number;
};
```

DaaS adapters reject `pickups.length > 1` and `payAtCounter` (capabilities false) so the dispatcher never sends unsupported jobs.

## 6. Synchronized legs (fallback)

1. Per leg: `ready_by`, provider pickup→dropoff ETA.
2. Target arrival window `[T, T+10 min]`; schedule each leg's creation so predicted arrival ∈ window.
3. Monitor ETAs; notify early if spread grows.
4. One merged tracking view.

## 7. Shop extras

### Discovery along the route

```sql
select s.*
from catalog.merchant_locations s
join catalog.merchants m on m.id = s.merchant_id
where m.kind in ('shop','florist','grocery','bakery')
  and st_dwithin(s.geom, :route_geom, :corridor_m)        -- e.g. 600 m
  and is_open(s.hours, :eta_at_stop, s.timezone);
```

- Partner shops with catalogs → exact items/prices; others → category requests with caps ("seasonal bouquet ≤ $25").
- Substitution rule per extra: same brand any size · similar item · skip · message me (2-min in-app approval, else fallback).
- In cluster runs the courier buys with the run card; on DaaS-only orders, extras need a separate shop leg (often not worth it — the Host says so).
- v1 restrictions: no alcohol, tobacco, vapes, cannabis, prescription/pharmacy-only items, gift cards; items > $50 need re-confirmation.

## 8. Tracking

- Cluster runs: driver-app location every 10–15 s while active; stop chips (🍰 picked up · 💳 paid · 🍽️ waiting · 🏠 7:28).
- DaaS: provider tracking URL + webhooks → normalized events.
- Proof of delivery: photo or PIN.

## 9. Edge cases

| Case | Handling |
|---|---|
| Food not ready at courier arrival | Courier waits up to N min (ruleset); ready ping; re-sequence remaining stops |
| Card declined at counter | Real-time auth log shows reason; ops can raise limit within policy or switch that leg to Connect settlement/virtual card |
| Price at counter > leg amount × 1.10 | Decline; courier calls ops; user asked if the difference exceeds tolerance |
| Courier unavailable mid-run | Ops reassigns within crew; else DaaS legs for remaining stops |
| Shop item unavailable | Substitution rule; refund difference |
| Severe weather | Pause cluster runs; DaaS only; widen windows |
