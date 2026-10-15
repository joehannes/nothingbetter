# ADR-0002 · PostgreSQL + PostGIS + pgvector as the primary store

- Status: Accepted (2026-10-15)

## Context
We need relational data (orders, ledger), geo queries (service areas, nearby merchants, route corridors), vector similarity (dishes, menus, memories) and a modest knowledge graph.

## Decision
PostgreSQL 17 with PostGIS and pgvector; one schema per module; Drizzle ORM; knowledge graph as `kn_entities` + `kn_relations` tables.

## Consequences
- + One operational database, transactional consistency, simple local dev.
- + Hybrid search (FTS + vector + geo) in one query.
- − Very large vector workloads may need a dedicated engine → revisit if p95 search > 150 ms.

## Alternatives
Separate graph DB (Neo4j), dedicated vector DB, MongoDB — more moving parts without v1 benefit.
