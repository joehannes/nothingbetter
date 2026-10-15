# AI/01 · AI Platform Architecture

> Status: Draft v1 · Model facts as of the Anthropic model table cached 2026-10-06 · Owner: AI Engineering

## 1. Goals

1. A **host-like conversation** that adapts tone and pace live (see [AI/04](04-conversation-engine-and-adaptive-dials.md)).
2. **Expert advice** from several specialist faculties (nutrition, natural medicine, culture, occasion, social, inspiration, logistics, safety) merged into one coherent recommendation.
3. **Grounded proposals**: every suggested dish exists on a live menu, fits every hard constraint, and can be delivered in time.
4. **Deterministic money and safety boundaries**: the LLM proposes; code validates and executes.
5. **Measurable**: every AI call is versioned, traced, costed and evaluated.

## 2. Layered design

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ L5 Presentation  Host/Orchestrator writes replies, chips, cards (tone ← dials)│
├─────────────────────────────────────────────────────────────────────────┤
│ L4 Decision      Meal-set composer (optimizer) + LLM rerank & rationale     │
├─────────────────────────────────────────────────────────────────────────┤
│ L3 Council       Faculties in parallel → Advice Cards → Synthesizer (code)  │
├─────────────────────────────────────────────────────────────────────────┤
│ L2 Context       Situation Brief · Dial engine · Ruleset engine · Memory     │
├─────────────────────────────────────────────────────────────────────────┤
│ L1 Grounding     Food graph (menus, dishes, nutrition) · Knowledge base ·  │
│                  Geo/ETA · Merchant status · Calendar/occasions             │
└─────────────────────────────────────────────────────────────────────────┘
```

The core algorithm (from the earlier investigation, extended):

**Intent → Context → Constraints → Council → Candidates → Combinations → Ranking → Explanation → Confirmation → Fulfillment → Feedback → Memory**

## 3. Agents and calls

| Component | Kind | Model & effort (default) | Output |
|---|---|---|---|
| **Host (orchestrator)** | Agent loop with client tools, streaming | `claude-opus-5-5`, effort `medium` (lowered to `low` when urgency ≥ 75) | Text stream + UI directives |
| **Faculties** (×8–12) | Single structured-output calls, parallel | `claude-opus-5-5`, effort `low` | `AdviceCard` JSON |
| **Synthesizer** | Deterministic code (`packages/domain`) | — | `ConstraintSet` + weights |
| **Meal-set composer** | Deterministic optimizer + one LLM rerank call | rerank: `claude-opus-5-5`, effort `medium` (`high` for groups ≥ 6 or ≥ 3 merchants) | Ranked `MealSet[]` + rationale |
| **Memory curator** | Async structured call after turns/orders | effort `low` | `MemoryProposal[]` |
| **Restaurant liaison parser** | Structured call on free-text/voice replies | effort `low` | `AvailabilityReply` |
| **Menu ingestion & enrichment** | Background, vision + structured outputs, Batch API | effort `medium` | `ParsedMenu`, `DishEnrichment` |
| **Support agent** | Agent loop with ops tools, human handoff | effort `medium` | Resolutions/escalations |

Model policy ([ADR-0004](../01-architecture/adr/ADR-0004-ai-platform-claude.md)): **one model (`claude-opus-5-5`) with tuned effort per route**. Before introducing a cheaper model for any route (e.g., `claude-haiku-5-5` at $0.10/$0.50 per MTok for bulk menu extraction), measure the same model at lower effort on that route's eval; any switch is an explicit owner decision. Exact model ID strings only — no date suffixes.

## 4. Claude API usage rules (Opus 5.5 specifics)

| Topic | Rule |
|---|---|
| Thinking | Always on for Opus 5.5 (adaptive). Do **not** send `{type:"disabled"}` or `budget_tokens` (400). Control depth with `output_config.effort`. Default effort is `medium` — set explicitly per route. |
| Thinking display | Default `omitted` (empty thinking text). We never show reasoning to users; leave default. Pass thinking blocks back unchanged in multi-turn tool loops. |
| Tool choice | Forced `tool_choice` (`any`/`tool`) returns 400 on Opus 5.5 → use `auto` + clear instruction, `strict: true` tool schemas, or structured outputs. |
| Structured outputs | `output_config: { format: zodOutputFormat(Schema) }` with `client.messages.parse()`; guard `parsed_output` null. |
| Prefill | Not supported (400). Use structured outputs or instructions. |
| Refusals | Always check `stop_reason === "refusal"` before reading content. Enable server-side fallbacks (`betas: ["server-side-fallback-2026-07-01"]`, `fallbacks: "default"`) on the Claude API. |
| Streaming | Stream the Host (user-facing) and any call with large `max_tokens`. For streamed requests with client tools set `eager_input_streaming: true` and validate tool inputs against the Zod schema before executing. |
| Mid-conversation system messages | Append `{ role: "system", content }` after a user message for dial changes, situation updates, safety notices — preserves the cached prefix (supported on Opus 5.5). |
| Per-message effort | Beta `mid-conversation-output-config-2026-07-01` lets us change effort mid-session without cache reset (urgency spikes). |
| Prompt caching | Order: `tools` → `system` → `messages`. Freeze system prompts and tool lists (no timestamps/IDs). Cache breakpoints on: Host system prompt; each faculty prompt + its knowledge pack. Verify `usage.cache_read_input_tokens` > 0 in tests. |
| Compaction | Beta `compact-2026-01-12` only if a session exceeds ~150K tokens (rare); cross-session memory is ours ([AI/05](05-memory-avatars-and-personalization.md)). |
| Batch | Nightly enrichment, memory consolidation, eval runs via Message Batches (50 % cost); results keyed by `custom_id`. |
| Errors | Catch typed SDK errors most-specific-first (`RateLimitError` → `APIStatusError` → `APIConnectionError`); SDK retries 2× by default. |
| Tool JSON | Always `JSON.parse`/schema-validate tool inputs; never string-match serialized inputs. |
| Data | Minimize PII in prompts: avatar ids + needed attributes; no phone/email/address lines unless required by the tool. |

### Faculty call (shape)

```ts
import Anthropic from "@anthropic-ai/sdk";
import { zodOutputFormat } from "@anthropic-ai/sdk/helpers/zod";
import { AdviceCardSchema } from "@nb/contracts/ai";

const client = new Anthropic();

export async function runFaculty(faculty: FacultySpec, brief: SituationBrief) {
  const res = await client.messages.parse({
    model: "claude-opus-5-5",
    max_tokens: 4000,
    output_config: { effort: faculty.effort ?? "low", format: zodOutputFormat(AdviceCardSchema) },
    system: [
      { type: "text", text: faculty.systemPrompt },                       // frozen, versioned
      { type: "text", text: faculty.knowledgePack, cache_control: { type: "ephemeral" } },
    ],
    messages: [{ role: "user", content: renderBriefForFaculty(faculty.id, brief) }],
  });
  if (res.stop_reason === "refusal" || !res.parsed_output) return emptyAdvice(faculty.id, res);
  return res.parsed_output;
}
```

### Host turn (shape)

```ts
const stream = client.beta.messages.stream({
  model: "claude-opus-5-5",
  max_tokens: 8000,
  betas: ["server-side-fallback-2026-07-01"],
  fallbacks: "default",
  output_config: { effort: dials.urgency >= 75 ? "low" : "medium" },
  system: [{ type: "text", text: HOST_SYSTEM_PROMPT, cache_control: { type: "ephemeral" } }],
  tools: HOST_TOOLS,                       // strict Zod-derived schemas, frozen order
  messages: [
    ...history,
    { role: "user", content: userTurn },
    { role: "system", content: renderLiveContext(brief, dials) }, // dials, time, table; changes each turn
  ],
});
```

(Exact SDK bindings are verified against `@anthropic-ai/sdk` at implementation time; this is the intended request shape.)

## 5. Council orchestration

```text
SituationBrief ─▶ Ruleset engine ─▶ active faculties + hard constraints + weight presets
                                   │
            ┌──────────┬───────────┼────────────┬────────────┬──────────┐
            ▼          ▼           ▼            ▼            ▼          ▼
         Safety*   Nutrition   NaturalMed   Culture/      Social/   Inspiration
        (always)   (if health  (if lens on) Occasion      Group     (if dial ≥ 40)
                    or goals)                (if any)    (≥ 2 seats)
            └──────────┴───────────┴────────────┴────────────┴──────────┘
                                   ▼
                       Synthesizer (deterministic)
              priority: Safety > Hard constraints > Medical (opt-in)
                 > Religious/cultural rules > Time & budget > Taste
                 > Health boosts > Inspiration boosts
                                   ▼
                         ConstraintSet + ScoreWeights
```

- Fan-out in parallel with a soft deadline (2.5 s). Late faculties are dropped for this turn except **Safety Sentinel**, which is required for any proposal.
- Faculty outputs are cached per `(brief.hash, faculty, version)` for the session.
- Faculties never see other faculties' outputs (independence → less groupthink); the Synthesizer and the rerank call see all.

### `AdviceCard` (contract)

```ts
type AdviceCard = {
  faculty: FacultyId;
  hardConstraints: { code: string; seat?: AvatarId; reason: string }[];   // only Safety may add new hard constraints
  boosts:   { target: TagOrDish; seat?: AvatarId; weight: number; why: string; evidence?: EvidenceLevel }[];
  cautions: { target: TagOrDish; seat?: AvatarId; severity: "info" | "warn" | "block"; why: string; evidence?: EvidenceLevel }[];
  ideas:    { title: string; dishes: CanonicalDishId[]; story?: string; seat?: AvatarId }[];
  questions?: { text: string; infoGain: number }[];   // what this faculty would like to know
  confidence: number;                                 // 0..1
};
```

## 6. Orchestrator tools

| Tool | Purpose | Side effects |
|---|---|---|
| `get_situation()` | Current Situation Brief + dials | none |
| `set_table({avatarIds})` | Confirm who's eating | updates session |
| `ask_user({question, chips[], multi})` | Ask the single highest-info-gain question with tappable chips | renders UI |
| `consult_council({focus?})` | Run active faculties for the current brief | AI calls |
| `search_food_graph({query, filters, near, openAt, limit})` | Candidate dishes/restaurants (hybrid search) | none |
| `compose_meal_sets({constraintSetId, seats, budget, maxMerchants, deliverBy})` | Optimizer + rerank → `MealSet[]` | AI call |
| `quote_logistics({mealSetId})` | Dispatch modes, fees, ETA, route | provider quotes (cached) |
| `show_proposals({sets, layout})` | Render cards / Table view | renders UI |
| `start_availability_check({basketId})` | Only after the user taps "Check & hold" | messages merchants |
| `propose_memory({statements[]})` | Suggest remembering facts | needs user confirmation for hard facts |
| `set_dials({urgency?, inspiration?, directness?, scenic?, reason})` | Model-proposed dial nudge (bounded ±15 per turn; user overrides win) | updates dials |
| `handoff_to_human({reason})` | Escalate to support | creates ticket |

Tools that spend money **do not exist** for the Host. Checkout is a user action in the UI; the server validates the basket deterministically.

## 7. Meal-set composition

1. **Candidates:** for each seat, top-N dishes from open merchants within delivery feasibility, filtered by hard constraints (SQL), scored by `seatFit = Σ wᵢ·featureᵢ` (taste match, dish rating, novelty bonus × inspiration dial, health fit × health emphasis, cultural fit, travel score, price fit).
2. **Combinations:** beam search over merchant sets (max merchants from ruleset/dials, usually 1–3), assigning dishes to seats; objective = Σ seatFit − λ_price·cost − λ_eta·lateness − λ_legs·extraLegs + λ_share·shareability (for group occasions).
3. **Diversity:** return 3–5 sets that differ meaningfully (one-merchant simple, best-fit, budget, adventurous).
4. **Rerank & rationale:** one LLM call sees sets + Advice Cards and returns ordering + 1–2 sentence "why" per set and per seat, in the session's tone and language.
5. **Validation:** deterministic re-check of hard constraints, prices, opening hours, ETA before rendering.

## 8. Context management

- **Situation Brief** (structured, ≤ 1.5K tokens) is the shared context; recomputed on signals (place change, table change, calendar tick).
- History sent to the Host: last 12 turns verbatim + a rolling session summary; proposals referenced by id.
- Avatar context: only seats at the table; hard constraints always; top-K preferences by relevance; ≤ 5 memories retrieved by embedding similarity to the brief.

## 9. Cost & latency budget (targets per session, Phase 1)

| Item | Budget |
|---|---|
| Host turns | ≤ 6 per ordering session (express: ≤ 2) |
| Council runs | ≤ 2 per session (cached thereafter) |
| Rerank calls | ≤ 2 |
| Cache read ratio on Host/faculties | ≥ 70 % of input tokens |
| End-to-end proposal latency | p50 ≤ 5 s |

Cost per completed order is measured from `ai.ai_runs` (tokens × current rates incl. cache reads) and reviewed weekly.

## 10. Failure modes & degradations

| Failure | Behavior |
|---|---|
| Claude unavailable / rate-limited | Fallbacks; then "classic mode": filters + search + saved favorites, no council |
| Faculty times out | Proceed without it (except Safety) and note internally |
| Optimizer finds no feasible set | Host explains the binding constraint ("Nothing halal open past 10 pm within 30 min — widen to 45 min?") |
| Low-confidence menu data | Exclude from proposals for seats with relevant hard constraints |
| Prompt injection in merchant/menu text | Treated as data inside delimited fields; structured outputs; no tool can act on it directly |
