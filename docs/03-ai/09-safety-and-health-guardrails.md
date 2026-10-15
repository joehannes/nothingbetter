# AI/09 · Safety & Health Guardrails

> Status: Draft v1 · Owner: AI Safety + Medical/Nutrition advisors · Sev-1 = any hard-constraint violation reaching a diner.

## 1. Defense in depth

```text
Layer 1  Data        hard constraints are user-confirmed; allergens only "declared" are facts
Layer 2  Retrieval   SQL filters drop violating items before any LLM sees them
Layer 3  Council     Safety Sentinel adds blocks/cautions; required for proposals
Layer 4  Composer    deterministic re-check of every item × seat before rendering
Layer 5  Checkout    validator re-checks basket vs constraints, prices, hours, address
Layer 6  Merchant    allergy notes sent in the firm confirmation ("SEVERE SESAME ALLERGY — seat Mia")
Layer 7  Feedback    "Something wrong?" → incident pipeline
```

## 2. Hard rules (code-enforced)

1. An item is offered to a seat with an **anaphylaxis/severe allergy** only if the merchant has **declared** it free of that allergen *and* acknowledged cross-contact handling; otherwise it's excluded.
2. Religious/diet rules: merchant-level declarations (e.g., halal-certified) or item labels; never inferred from dish names alone.
3. Minors: no alcohol, no caffeine-heavy drinks for children, spice defaults mild, choking-hazard caution for under-4s, no honey for infants under 12 months.
4. Pregnancy (opt-in): exclude high-risk items (raw/undercooked fish & meat, unpasteurized cheeses, high-mercury fish, alcohol) unless the user explicitly overrides for non-allergy cautions.
5. Interactions: if medications provided, cautions for known food–drug interactions (e.g., grapefruit with some statins/calcium-channel blockers, vitamin-K-rich foods with warfarin — consistency message, licorice with blood pressure). Never dosing advice.
6. No medical claims: blocked phrases list ("cures", "treats", "prevents disease", "detox your liver", "boosts immunity guaranteed") enforced by output check; rewrite or drop.

## 3. Sensitive conversation protocols

| Trigger | Response |
|---|---|
| Medical emergency cues (chest pain, can't breathe, severe allergic reaction) | Stop ordering flow; advise calling emergency services (911 US/CA); show button; log incident |
| Self-harm / suicidal ideation | Supportive, non-judgmental message; 988 Suicide & Crisis Lifeline (US & Canada); offer to continue later |
| Disordered eating cues | No calorie/weight talk for the session; gentle tone; resources (e.g., NEDA in US, NEDIC in Canada) if appropriate |
| Persistent illness symptoms (fever > 3 days, dehydration) | Suggest seeing a clinician; continue with gentle food options |
| Requests outside scope (medical diagnosis, legal, unrelated tasks) | Briefly decline, redirect to food & hosting |
| Abuse/harassment toward merchants or couriers | De-escalate; flag to ops |

## 4. Natural-medicine policy (preferred lens, safely)

- Opt-in lens; default suggested at onboarding with a one-line explanation.
- Every statement has an evidence level (A–D) and uses the phrasing table ([AI/06 §4](06-knowledge-base-culture-health-seasons.md#4-evidence-levels-health--natural-medicine)).
- Herbs/supplements only as **culinary amounts** or common teas from shops; no supplement dosing; no "instead of your medication".
- Pregnancy, breastfeeding, children, anticoagulants, immunosuppressants, chemotherapy → conservative mode (fewer herbal suggestions, explicit "check with your clinician").

## 5. Fairness & respect

- No stereotyping by heritage, gender, age, body; cultural suggestions only from declared heritage or explicit curiosity.
- Body-neutral language; no unsolicited weight comments.
- Kid-related content is caregiver-directed.

## 6. AI-specific safeguards

- Structured outputs for all faculties; free text only in Host replies, which pass an output checker (blocked claims, allergy contradictions, price/ETA mismatch vs proposal data).
- Refusal handling: `stop_reason === "refusal"` → server-side fallback; if still refused, the Host says it can't help with that request and offers a normal ordering path.
- Prompt injection: merchant replies, menus, user-provided links are wrapped as data; tools only accept validated ids.
- Logging: safety events with redacted content for review; weekly review of samples by the safety owner.

## 7. Advisory board & review

- Registered dietitian (US) and Canadian RD, licensed naturopathic doctor and/or TCM practitioner, allergist consultant (allergen policy), cultural consultants per launch community.
- Quarterly review of guardrails, knowledge, incidents, and eval results.
