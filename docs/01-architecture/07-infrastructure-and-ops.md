# 07 · Infrastructure & Operations

> Status: Draft v1

## 1. Environments

| Env | Purpose | Data | Providers |
|---|---|---|---|
| `local` | Developer machines | Seeded fixtures | Simulators + sandboxes (Stripe test, DoorDash Drive sandbox, WhatsApp test number) |
| `dev` | Shared integration | Synthetic | Sandboxes |
| `staging` | Release candidate, E2E, evals | Synthetic + anonymized samples | Sandboxes; production-like config |
| `prod-us` | US users | Real | Live |
| `prod-ca` | Canadian users (Phase 3) | Real (in Canada) | Live |

## 2. Local development

```bash
pnpm install
docker compose up -d          # postgres (postgis+pgvector), valkey, minio, mailpit
temporal server start-dev     # Temporal dev server + UI
pnpm db:migrate && pnpm db:seed   # schema + demo metro, merchants, menus, knowledge
pnpm dev                      # platform API, workers, app (Vite) concurrently
pnpm tauri dev                # desktop shell around the app
pnpm sim:restaurants          # fake merchants answering WhatsApp/SMS checks
pnpm sim:couriers             # fake dispatch provider emitting status/positions
```

`.env.example` lists every variable; `packages/config/env.ts` validates them with Zod at boot.

## 3. Cloud topology (AWS, `prod-us`)

```text
CloudFront (site, app static, OpenAPI docs)
ALB ──▶ ECS Fargate: platform-api (2–6 tasks), realtime (2+), voice-gateway (2+, autoscale on sessions)
        ECS Fargate: temporal-workers (order, liaison, dispatch, ingestion queues)
RDS PostgreSQL 17 (Multi-AZ) + read replica (Phase 2)
ElastiCache Valkey · S3 (menus, media) · Secrets Manager · KMS
Temporal Cloud namespace per env
```

IaC: OpenTofu modules in `infra/modules/*`, envs in `infra/envs/*`. All changes via PR + plan output.

## 4. CI/CD

| Pipeline | Steps |
|---|---|
| PR | Biome lint, typecheck, unit + integration tests (Testcontainers), contract tests vs provider fakes, dependency-cruiser, OpenAPI diff, AI eval smoke (≤ 50 cases, budget-capped) |
| Main | All of the above + E2E (Playwright) on staging + full AI eval suite nightly |
| Release (backend) | Build images → staging → canary 10 % → prod |
| Release (Tauri) | `tauri-action` matrix (macOS universal, Windows, Linux AppImage/deb, iOS, Android); signing & notarization; updater manifest; staged rollout |
| Knowledge | Validate `knowledge/**/*.yaml` against JSON Schemas; require reviewer role label for health/natural-medicine changes |

## 5. Observability & SLOs

| SLO | Target | Alert |
|---|---|---|
| Ordering API availability | 99.9 % / 30 d | Burn rate 2 %/1 h |
| Assistant first token p95 | ≤ 2.5 s | p95 > 3 s for 10 min |
| Availability-check answer p90 | ≤ 5 min | > 7 min for 30 min |
| Dispatch assignment p90 | ≤ 10 min | > 15 min |
| Webhook processing lag | ≤ 30 s | > 2 min |
| AI refusal/fallback rate | Baseline + 2σ | Spike |
| Hard-constraint violation | 0 | Page immediately |

Dashboards: funnel (session → proposal → basket → check → pay → delivered), per-merchant response times, per-provider dispatch performance, AI cost per order, cache hit rate.

## 6. Runbooks

| Scenario | First actions |
|---|---|
| WhatsApp template paused/rejected | Switch affected legs to SMS/voice channel; submit fixed template; check quality rating |
| Meta/Twilio outage | Liaison auto-falls back to next channel; ops dashboard shows manual call queue |
| DaaS provider outage | Nash dispatch strategy failover; disable provider flag; notify affected users with new ETAs |
| Stripe Issuing authorization declines | Check spending controls/category mismatch; issue replacement single-use card; ops can approve via real-time auth override |
| Anthropic API errors/rate limits | Retries with backoff; server-side fallback models; degrade to "classic mode" (search + filters, no council) |
| Allergen incident | Sev-1: freeze affected item mapping, contact user, preserve logs, root cause in 24 h |
| Data deletion request | DSAR tool; purge across DB, object store, search index, backups policy noted |

## 7. Support operations

- Ops console: live orders, leg statuses, merchant message threads, manual override (mark ready, re-dispatch, refund within limits), call merchant (click-to-call).
- Human-in-the-loop queue: low-confidence menu parses, unanswered availability checks, safety escalations.
- Staffing in Phase 1: one on-call operator during service hours (11:00–22:00 local).
