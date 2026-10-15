# ADR-0003 · Modular monolith + Temporal for workflows

- Status: Accepted (2026-10-15)

## Context
Orders span minutes to hours, involve several external parties (merchants via messaging, payment provider, couriers) with timeouts, retries, human steps and compensations.

## Decision
A Fastify modular monolith (`services/platform`) with strict module boundaries, plus Temporal workflows (`services/workers`) for: OrderSaga, AvailabilityCheck, DispatchRun, MenuIngestion, Enrichment, MemoryConsolidation. Domain events via transactional outbox.

## Consequences
- + Durable state machines with timers and signals; crash-safe; visible in Temporal UI.
- + Easy to split modules into services later.
- − Team must learn Temporal's determinism rules (no I/O in workflow code; activities only).

## Alternatives
Hand-rolled job queues (BullMQ) + state tables (error-prone), AWS Step Functions (less ergonomic for TS, vendor lock), Inngest/Restate (viable; Temporal chosen for maturity).
