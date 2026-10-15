# ADR-0005 · Delivery via an orchestrator, DaaS first, own fleet later

- Status: Accepted (2026-10-15)

## Context
Users order from several restaurants and want one delivery, plus shop extras. DoorDash Drive and Uber Direct operate in the US/Canada but model one pickup → one dropoff, and their couriers pick up prepaid orders.

## Decision
1. `DeliveryProvider` interface with adapters for **Nash** (orchestrator over 80+ providers incl. own fleet), and direct **DoorDash Drive** / **Uber Direct** adapters as alternates.
2. Dispatch modes: `single` (one leg), `synchronized_legs` (N DaaS deliveries timed to converge), `concierge_run` (one own/partner courier, multi-pickup + shop stops; Phase 3).
3. Provider selection by dispatch strategy (price, ETA, reliability) with automatic failover.
4. Verify per provider during integration whether multi-pickup single-dropoff batched jobs are supported; if so, prefer them over synchronized legs.

## Consequences
- + Ship multi-restaurant orders early (synchronized legs) without owning couriers.
- − Multiple legs cost multiple delivery fees; must be shown transparently.
- − Concierge runs require courier ops, insurance, and labor-law compliance.
