# AI/11 · Evaluation & Quality

> Status: Draft v1 · Suites live in `evals/` · Run: PR smoke (budget-capped), nightly full, pre-release gate.

## 1. Eval suites

| Suite | Cases (v1) | Grading | Gate |
|---|---|---|---|
| **Hard-constraint safety** | 400 (allergies × cuisines, diet/religious rules, minors, pregnancy, interactions) | Deterministic checks on proposals | 100 % pass (blocker) |
| **Safety conversations** | 150 (emergency, self-harm, disordered eating, out-of-scope, jailbreaks, injection in merchant text) | Rubric (LLM judge calibrated vs human labels) + regex for banned claims | ≥ 98 %, zero criticals |
| **Journeys** (from [personas](../00-product/03-personas-and-journeys.md)) | 120 multi-turn scripted users | Task success, turns-to-order, constraint adherence, tone fit by lane | Regression ≤ 2 pts |
| **Dial adaptation** | 200 situations | Lane chosen vs label; message length; questions count | ≥ 90 % lane agreement |
| **Faculty quality** | 30–60 per faculty | Expert rubric (RD, naturopath, cultural consultants); evidence-level correctness | Advisor sign-off |
| **Meal-set quality** | 150 group scenarios | Coverage of seats, budget, diversity of sets, rationale accuracy | Regression ≤ 2 pts |
| **Menu extraction** | 50 golden menus | Item/price F1, modifier accuracy | F1 ≥ 0.95 |
| **Liaison reply parsing** | 300 real-style replies (EN/ES, typos, voice-note transcripts) | Exact match on availability/time/substitutions | ≥ 97 % |
| **Voice** | 300 utterances | WER, slot accuracy | Vendor selection + regression |
| **Multilingual** | 100 EN/ES/FR-CA | Fluency, correct language, dish naming | ≥ 95 % |

## 2. Judges

- LLM-as-judge with written rubrics; calibrated against ≥ 100 human-labeled examples per rubric (agreement ≥ 85 %).
- Judges never grade their own route's output with the same prompt version; rubric changes are versioned.
- Deterministic checks preferred wherever possible (constraints, counts, lengths, prices).

## 3. Simulated users

- Persona-conditioned simulators (scripted goals, hidden constraints, impatience levels) drive multi-turn journeys against a seeded metro (fake merchants, menus, simulators for liaison and dispatch).
- Simulators are separate prompts with fixed seeds where supported; transcripts saved for diffing.

## 4. Online metrics

| Metric | Use |
|---|---|
| Proposal acceptance, edits per accepted set | Recommendation quality |
| Lane override rate | Dial quality |
| "Too long / too many questions" feedback | Tone |
| Dish "loved" rate by seat | Personalization |
| Safety notices per 1K sessions, incident count | Safety |
| Cost per completed order, cache-read ratio | Efficiency |

## 5. Process

- Every prompt/ruleset/model change ships with an eval diff in the PR.
- Weekly quality review: worst 20 sessions (by feedback/overrides), safety samples, cost outliers.
- Before switching any route's model or effort: run that route's suite; cost per *completed task*, not per request.
