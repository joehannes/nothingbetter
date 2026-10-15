# CLAUDE.md — guidance for coding agents

This repo is **docs-first**. Before writing code, read `docs/README.md`, then the doc(s) for the area you're touching. If code and docs disagree, raise it; update the doc (or add an ADR) in the same PR as the code.

## Where to start

1. Implement milestones in order from `docs/05-delivery-plan/02-mvp-scope-and-backlog.md` (M0 → M6). Reference story IDs (e.g., `NB-M3-04`) in commits and PRs.
2. Scaffold exactly as in `docs/01-architecture/03-monorepo-layout.md`.
3. Contracts first: add/modify Zod schemas in `packages/contracts` before endpoints or UI.

## Non-negotiables

- **Hard constraints are sacred.** No code path may present, add to a basket, or order an item that violates a present seat's confirmed allergy/diet/religious/medical constraint. Keep the property tests in `docs/05-delivery-plan/03-testing-strategy.md` §3 green.
- **The AI never moves money.** Payment, merchant payment and dispatch happen only through deterministic validators and workflows. Host tools must not include payment actions.
- **No sensitive inference.** Never infer culture, religion, ethnicity or health from names, voice, location or orders (ADR-0010).
- **Merchant messaging requires opt-in evidence** and respects quiet hours (WF/02).
- **No scraping or automated ordering on marketplaces' consumer apps** (Uber Eats, DoorDash, Grubhub, PedidosYa).
- **Health/natural-medicine wording** follows the evidence-level table in `docs/03-ai/06` §4; no disease claims.

## Coding conventions

- TypeScript strict, ESM, Node 24 LTS; Biome for lint/format; pnpm + Turborepo.
- Money: integer minor units + ISO currency. Time: UTC `timestamptz`; IANA zones on places/merchants.
- IDs: prefixed ULIDs (`usr_`, `av_`, `ord_`, `leg_`, `rst_`…).
- Module boundaries: modules call each other via public interfaces/events, never each other's tables.
- Pure domain logic (dials, synthesizer, composer, pricing, state machines, routing) lives in `packages/domain` with no I/O.
- Provider SDKs only inside `packages/providers/*` behind interfaces (`DeliveryProvider`, `MerchantChannel`, `SpeechToText`, `TextToSpeech`, `PosAdapter`).
- Temporal workflow code is deterministic; all I/O in activities.

## Claude API usage (see docs/03-ai/01)

- SDK: `@anthropic-ai/sdk`. Default model `claude-opus-5-5` on every route; set `output_config.effort` explicitly per route (default on Opus 5.5 is `medium`).
- Thinking is always on for Opus 5.5 — never send `thinking: {type: "disabled"}` or `budget_tokens`.
- No forced `tool_choice` (`any`/`tool`) — use `auto` + instructions, `strict: true` tools, or structured outputs (`client.messages.parse` + `zodOutputFormat`).
- No assistant prefill. Check `stop_reason === "refusal"`; enable server-side fallbacks (`fallbacks: "default"`, beta `server-side-fallback-2026-07-01`).
- Prompt caching: frozen system prompts/tool lists; dynamic context as mid-conversation `role: "system"` messages; assert `cache_read_input_tokens > 0` in tests.
- Exact model ID strings only (no date suffixes). Don't switch a route's model without running its eval suite; cheaper-model decisions belong to the owner.
- Log every call to `ai.ai_runs` (prompt version, ruleset version, tokens, cost, latency, stop reason).

## Knowledge & prompts

- Prompts: `knowledge/templates/prompts/**` (front matter + body). Rulesets: `knowledge/rulesets/**` (JSONLogic conditions). Validate with JSON Schemas in CI.
- Health and cultural content changes need advisor review labels (`review:health`, `review:culture`).

## Definition of done (per story)

- Acceptance criteria met; unit/integration tests; relevant AI eval suite run with diff in PR.
- Accessibility (axe clean) for UI; i18n strings (EN + ES) via ICU messages.
- Telemetry (traces, AI run logs) and docs updated.
