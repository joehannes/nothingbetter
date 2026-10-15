# 03 · Monorepo Layout

> Status: Draft v1 · The scaffold to create in Milestone M0 ([backlog](../05-delivery-plan/02-mvp-scope-and-backlog.md)).

```text
nothingbetter/
├── apps/
│   ├── app/                     # React SPA (eater experience) — runs in Tauri and on the web
│   │   ├── src/
│   │   │   ├── routes/          # TanStack Router file routes
│   │   │   ├── features/
│   │   │   │   ├── conversation/   # chat + voice UI, proposal cards, chips
│   │   │   │   ├── table/          # Table view (seats, drag dishes)
│   │   │   │   ├── basket/         # multi-leg basket, route preview
│   │   │   │   ├── checkout/
│   │   │   │   ├── tracking/
│   │   │   │   ├── avatars/        # household & people
│   │   │   │   ├── memory/         # "What I know about you"
│   │   │   │   ├── places/         # addresses, GPS
│   │   │   │   └── settings/       # dials defaults, lenses, voice
│   │   │   ├── platform/        # adapters: tauri vs web (geo, audio, push, storage)
│   │   │   └── i18n/
│   │   └── vite.config.ts
│   ├── tauri/                   # Tauri 2 shell (src-tauri/), mobile projects (gen/apple, gen/android)
│   │   └── src-tauri/
│   │       ├── src/             # Rust: commands, nb-audio plugin, permission handlers
│   │       ├── capabilities/    # default.json, mobile.json
│   │       ├── Info.plist       # NSMicrophoneUsageDescription, NSLocationWhenInUseUsageDescription
│   │       └── tauri.conf.json
│   ├── site/                    # Astro marketing site (SEO, city pages, partner signup)
│   ├── partner/                 # Restaurant & shop console (React SPA)
│   └── ops/                     # Internal ops/support console (React SPA)
├── services/
│   ├── platform/                # Fastify modular monolith (API + module code)
│   │   └── src/modules/{identity,people,conversation,ai,knowledge,catalog,geo,order,
│   │                      liaison,payments,dispatch,notifications,ops}/
│   ├── workers/                 # Temporal workers (workflows + activities)
│   └── voice-gateway/           # Streaming STT/TTS gateway (WS now, LiveKit agent later)
├── packages/
│   ├── contracts/               # Zod schemas: API DTOs, events, AI structured outputs
│   ├── api-client/              # Generated OpenAPI client + TanStack Query hooks
│   ├── ui/                      # Design system components + tokens
│   ├── domain/                  # Pure domain logic: pricing, state machines, scoring, dials
│   ├── ai-kit/                  # Claude client wrapper, prompt registry, faculty runner
│   ├── knowledge-kit/           # Loaders/validators for knowledge/ YAML
│   ├── providers/               # Adapters: stripe, nash, doordash, uber, whatsapp, twilio, mapbox, pos/*
│   ├── db/                      # Drizzle schema, migrations, seed
│   ├── config/                  # Shared tsconfig, biome, env schema
│   └── testing/                 # Fixtures, simulators (restaurant, courier, provider fakes)
├── knowledge/                   # Versioned knowledge: rulesets, cultures, occasions, templates
├── evals/                       # AI eval suites (golden conversations, safety sets, judges)
├── infra/                       # OpenTofu modules, environments
├── docs/                        # You are here
├── .github/workflows/           # CI: lint, typecheck, test, build, evals, tauri builds
├── CLAUDE.md                    # Guidance for coding agents
├── package.json / pnpm-workspace.yaml / turbo.json / biome.json
└── .env.example
```

## Dependency rules

```text
apps/*  ──▶ packages/{api-client, ui, contracts, domain}
services/* ──▶ packages/{contracts, domain, ai-kit, knowledge-kit, providers, db}
packages/domain ──▶ packages/contracts            (no I/O, no SDKs)
packages/providers ──▶ packages/contracts         (each adapter behind an interface)
apps/* never import services/* or packages/providers.
```

Enforce with `dependency-cruiser` (or Biome's import rules) in CI.

## Naming & conventions

- Packages: `@nb/<name>`; modules: kebab-case folders; TS types `PascalCase`; Zod schemas `xxxSchema`.
- Every money field: `{ amountMinor: number; currency: 'USD' | 'CAD' }`.
- Every timestamp: ISO 8601 UTC string in contracts; `timestamptz` in DB.
- IDs: prefixed ULIDs (`usr_…`, `av_…`, `ord_…`, `leg_…`, `rst_…`) for log readability.
- Events: `domain.entity.verb` past tense (`order.leg.confirmed`).
