# 02 · Tech Stack

> Status: Draft v1 · Pin exact versions in `package.json` / `Cargo.toml` at scaffold time; prefer versions released ≥ 2 weeks earlier.

## 1. Client

| Concern | Choice | Notes |
|---|---|---|
| Shell | **Tauri 2** (Rust) | Desktop (macOS, Windows, Linux) + mobile (iOS, Android) from one project |
| UI | **React 19 + TypeScript (strict)** | One SPA shared by Tauri and web |
| Build | **Vite** | Tauri's `beforeDevCommand` / `frontendDist` point at the SPA |
| Routing | **TanStack Router** | Typed routes, search-param state |
| Server state | **TanStack Query** | Generated OpenAPI client underneath |
| Local UI state | **Zustand** | Small, explicit stores (dials override, voice state, basket draft) |
| Styling | **Tailwind CSS v4** + CSS variables design tokens | Tokens from [design system](../04-design/02-design-system.md) |
| Primitives | **Radix UI** primitives, own component library in `packages/ui` | Accessible by default |
| Motion | **Motion** (Framer Motion) + **Rive** for the Host orb / mascot | Respect `prefers-reduced-motion` |
| Forms & validation | React Hook Form + **Zod** (shared schemas from `packages/contracts`) | |
| i18n | **FormatJS / ICU MessageFormat** | EN, ES, FR-CA at launch |
| Maps | **MapLibre GL JS** with Mapbox (or MapTiler) tiles | Route + stops visualization |
| Audio | Web Audio API + AudioWorklet; **Silero VAD** via onnxruntime-web | Native Rust capture fallback (cpal) for Linux |
| Offline/PWA | Service worker (web), Tauri store plugin (desktop/mobile) | Cache last session, addresses, avatars |
| Testing | Vitest, Testing Library, Playwright (web), Tauri WebDriver (desktop smoke) | |

### Tauri plugins (initial set)

| Plugin | Use |
|---|---|
| `geolocation` | GPS on iOS/Android (needs `NSLocationWhenInUseUsageDescription`; Android location permissions auto-added) |
| `notification` | Local + push presentation |
| `deep-link` | `nothingbetter://` links (group invites, order links) |
| `store` | Small local persistence |
| `stronghold` or OS keychain | Refresh tokens |
| `opener` | External links (tracking URLs) |
| `os`, `process`, `updater` | Diagnostics, desktop auto-update |
| Custom `nb-audio` (Rust) | Native mic capture + optional on-device STT (whisper.cpp) where the WebView path is unreliable |

**Microphone permissions:** macOS/iOS need `NSMicrophoneUsageDescription` in `src-tauri/Info.plist` (without it capture is silently denied); Android needs `RECORD_AUDIO`; on Linux, WebKitGTK denies `getUserMedia` unless the app handles the user-media permission request — wire a handler that grants it for our own origin only, or use the native `nb-audio` path.

## 2. Backend

| Concern | Choice | Notes |
|---|---|---|
| Runtime | **Node.js 24 LTS**, TypeScript strict, ESM | Same language as the client; shared contracts |
| HTTP | **Fastify** + `@fastify/websocket`, `@fastify/rate-limit`, `@fastify/helmet` | OpenAPI from Zod via `fastify-type-provider-zod` |
| Contracts | **Zod** schemas in `packages/contracts` → OpenAPI 3.1 → generated client (`openapi-typescript` + `openapi-fetch`) | Single source of truth |
| Workflows | **Temporal** (TypeScript SDK), Temporal Cloud in production | Sagas, timers, retries, human-in-the-loop signals |
| Database | **PostgreSQL 17** + **PostGIS** + **pgvector** | One database, schemas per module |
| ORM / migrations | **Drizzle ORM** + drizzle-kit | SQL-first, typed |
| Cache / pub-sub | **Valkey/Redis** | Rate limits, session cache, realtime fan-out |
| Object storage | S3-compatible (AWS S3 or Cloudflare R2) | Menu files, images, voice notes (short retention) |
| Search | Postgres FTS + pgvector hybrid; Typesense later if needed | Geo filter via PostGIS |
| Embeddings | Voyage AI (Anthropic-recommended partner) or self-hosted open model — choose by eval | Dish/menu/memory embeddings |
| AI | **Anthropic Claude** via `@anthropic-ai/sdk` | See [AI/01](../03-ai/01-ai-platform-architecture.md) |
| Voice | Streaming STT (Deepgram or AssemblyAI), TTS (ElevenLabs or Cartesia); **LiveKit** (WebRTC + Agents) in Phase 2 | Vendor-neutral `SpeechProvider` interface |
| Messaging | **WhatsApp Cloud API** (direct or via Twilio as BSP), **Twilio** SMS + Voice | `MerchantChannel` interface |
| Payments | **Stripe** Payments, Issuing, Connect, Tax | |
| Delivery | **Nash** (orchestrator) and/or direct **DoorDash Drive** + **Uber Direct**; **Onfleet** or Nash own-fleet for concierge runs | `DeliveryProvider` interface |
| Geo | **Mapbox** Search/Geocoding, Directions, Matrix | Google Places optional for discovery (license limits) |
| POS | Square, Toast, Clover partner APIs (Phase 2) | `PosAdapter` interface |
| Email | Postmark or Amazon SES | Receipts |
| Push | APNs, FCM, Web Push | |

## 3. Platform & tooling

| Concern | Choice |
|---|---|
| Monorepo | **pnpm workspaces + Turborepo** |
| Lint/format | **Biome** (lint + format) for TS; `rustfmt` + `clippy` for Rust |
| Type checking | `tsc --noEmit` per package, project references |
| Testing | Vitest (unit/integration), Testcontainers (Postgres, Redis), Temporal test env, Playwright (E2E), k6 (load) |
| CI/CD | GitHub Actions; Tauri builds via `tauri-action`; code signing (Apple notarization, Windows signing) |
| IaC | OpenTofu (Terraform-compatible) |
| Hosting | AWS: ECS Fargate (api, workers, voice), RDS Postgres, ElastiCache, S3, CloudFront; Temporal Cloud |
| Observability | OpenTelemetry → Grafana (Tempo/Loki/Prometheus) or Honeycomb; Sentry; PostHog (product analytics, self-hosting option) |
| Secrets | AWS Secrets Manager; never in repo; `.env.example` only |
| Docs | This `docs/` tree; ADRs; OpenAPI published from CI |

## 4. Why not …

| Alternative | Reason not chosen (now) |
|---|---|
| Electron | Larger binaries and memory; Tauri requested and gives mobile targets |
| React Native for mobile | Two UI stacks; Tauri 2 mobile keeps one React codebase. Revisit if mobile performance or background location needs exceed Tauri. |
| Next.js for the app | App runs inside Tauri as a static SPA; SSR not needed there. Marketing SEO handled by Astro. |
| Microservices from day one | Overhead without scale; modular monolith + Temporal gives seams to split later |
| Kafka | Temporal + outbox + Redis cover v1 volumes |
| Separate graph DB | Relational tables + pgvector cover the knowledge graph at v1 scale |
