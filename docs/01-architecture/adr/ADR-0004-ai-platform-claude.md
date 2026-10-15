# ADR-0004 · Claude as the AI platform; one model, tuned effort per route

- Status: Accepted (2026-10-15)

## Context
The assistant needs strong multi-step reasoning (group meal composition under hard constraints), tool use, structured outputs, vision (menu photos), multilingual conversation and safety behavior.

## Decision
- Use Anthropic's Claude via the TypeScript SDK (`@anthropic-ai/sdk`).
- Default model for every route: **`claude-opus-5-5`** with **per-route effort** (`low` for faculties/classification, `medium` for the orchestrator, `high` for complex group planning). Effort default on Opus 5.5 is `medium` — set it explicitly.
- Enable server-side refusal fallbacks (`fallbacks: "default"` with beta `server-side-fallback-2026-07-01`) and always check `stop_reason`.
- Structured outputs (`output_config.format`) for faculties; `strict: true` tools; no forced `tool_choice` (unsupported on Opus 5.5).
- Prompt caching for stable system prompts, faculty instructions and knowledge packs.
- A cheaper model (e.g., `claude-haiku-5-5`) for high-volume background extraction is an **owner decision after evals**, not a default (measure "same model at lower effort" first).

## Consequences
- + One cache namespace, simpler evals, consistent behavior.
- + Effort is a per-route cost/latency lever.
- − Cost per session must be monitored (`ai.ai_runs`), with budgets per route.

## Alternatives
Multi-vendor routing (complexity, inconsistent safety); open-weight self-hosting (ops burden; possible later for embeddings/on-device STT).
