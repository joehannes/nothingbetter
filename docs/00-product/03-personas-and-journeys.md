# 03 · Personas & Journeys

> Status: Draft v1 · These journeys are the acceptance scenarios for AI evals and E2E tests ([testing](../05-delivery-plan/03-testing-strategy.md)).

## 1. Personas

| Persona | Snapshot | Jobs to be done | Default dials (U/I/D/S)* |
|---|---|---|---|
| **Carlos, household organizer** | 41, Miami, Cuban-American, wife Ana, kids Mia (8, sesame allergy) and Leo (13, picky), mother Rosa (72, low-sodium) visits Sundays | Weeknight dinner in 3 taps; Sunday family meals that work for Rosa; never an allergy slip | 60 / 30 / 60 / 30 |
| **Priya, busy professional** | 29, Toronto, vegetarian (Hindu tradition), back-to-back meetings | Desk lunch in 25 min, protein-forward, no fuss | 85 / 15 / 90 / 10 |
| **Jordan & Sam, the hosts** | 34 & 36, Brooklyn, anniversary, game nights, in-laws visiting | Make it special; mix cuisines; add flowers/drinks on the way | 25 / 80 / 30 / 85 |
| **Mei, curious eater** | 24, Vancouver, grad student, budget $20 | Try something new without a miss; learn the story | 40 / 90 / 50 / 70 |
| **Daniel, recovering** | 52, Houston, post-flu, mild hypertension, likes natural remedies | Comforting, warming, easy to digest; gentle guidance | 50 / 30 / 60 / 30 |
| **Aisha's circle** | 31, Dearborn, organizing iftar for 6 friends during Ramadan | Halal, served right at sunset, generous, shareable | 55 / 50 / 50 / 60 |

\* **U**rgency / **I**nspiration / **D**irectness / **S**cenic, 0–100 — see [AI/04](../03-ai/04-conversation-engine-and-adaptive-dials.md).

## 2. Journey A — "Work lunch, 25 minutes" (express lane)

Context: Tuesday 12:05, desktop app, calendar shows a meeting at 12:45, Priya's history = 70 % lunch orders under $18.

1. App opens on **Express** layout. Header chip: *"Lunch · 40 min until your 12:45"*.
2. Priya says (voice, push-to-talk): *"Something with protein, not heavy."*
3. Assistant (directness high, no small talk): *"Three picks, all vegetarian, here by 12:35:"* → 3 cards (paneer tikka bowl, black-bean burrito bowl, tofu bánh mì). Each shows ETA, price, protein estimate, one-line why.
4. Priya taps the paneer bowl → one-tap checkout with saved card (availability check runs in the background while she confirms; if POS-integrated, instant).
5. Tracking banner; no feedback prompt until next session (*"Paneer bowl yesterday: 👍/👎?"*).

**Rules exercised:** ≤ 1 question; ≤ 3 options; no stories; no upsell; effort `low`.

## 3. Journey B — "Family weeknight" (guided)

Context: Thursday 18:10, mobile, Carlos at home, household = Ana, Mia, Leo; Rosa not present.

1. *"Dinner for the four of you tonight?"* (assistant infers default table from household + time; one tap to confirm or change seats).
2. Carlos: *"Yeah, something everyone eats, Leo's been complaining."*
3. Assistant composes **table sets**: per-seat dishes from one or two restaurants. Mia's seat shows *"sesame-free (restaurant-verified)"* badge; Leo's seat shows a familiar dish he rated "loved".
4. Option A (one restaurant, $58, 35 min); Option B (two restaurants, $63, 40 min, two couriers arriving within 8 min of each other — stated plainly).
5. Carlos picks A, swaps Ana's dish via **Table view** drag.
6. Availability check with the kitchen (WhatsApp) → ✅ in 90 s → pay → delivered.
7. Post-meal: per-seat quick feedback; Leo's "meh" lowers that dish's score for Leo only.

## 4. Journey C — "Anniversary night in" (scenic)

Context: Saturday 17:00, calendar event "Our 5th 🥂", Jordan & Sam's avatars, budget soft $120.

1. Scenic layout, warmer tone: *"Five years — want to recreate something or make it new?"* with chips **Recreate our first date · Surprise us · A tasting journey**.
2. Jordan: *"Our first date was at a little Peruvian place that closed."*
3. Inspiration + Cultural faculties propose a **three-stop menu**: ceviche & lomo saltado from a Peruvian kitchen, a passion-fruit tart from a pastry shop, and a florist stop for a small bouquet on the route. Each card tells a 1–2 sentence story.
4. Logistics shows the route: 🍰 pastry (cold, first) → 💐 florist → 🍽️ Peruvian (hot, last) → 🏠. Mode: concierge run (Phase 3) or two synchronized legs (Phase 2) — the fee difference is shown.
5. Availability across 3 merchants (WhatsApp ✅, SMS ✅, POS ✅), then payment with authorized cap for the florist item.
6. Delivery at the requested time window 19:30–19:45 (scheduled).

## 5. Journey D — "Sick day" (care mode)

Context: Daniel says *"I'm getting over the flu, nothing heavy."* Health lens and natural-medicine lens are on (opt-in at onboarding).

1. Safety Sentinel checks red flags (fever > 3 days, breathing issues → suggest seeing a clinician; never diagnose).
2. Natural Medicine faculty (traditional framing): warming, easy-to-digest foods — ginger-scallion congee, chicken soup with garlic, miso soup; evidence notes like *"Ginger is traditionally used for nausea; some studies support this."*
3. Nutrition faculty flags sodium for his hypertension → proposes lower-sodium options and "ask for broth on the side".
4. Shop extras: ginger tea and honey from a pharmacy/grocery on the route (Phase 3), or a separate leg.
5. Tone: gentle, short, no upsell. Follow-up next day is opt-in only.

## 6. Journey E — "Iftar for six" (cultural + timing)

1. Aisha: *"Iftar for 6 on Friday, halal please."* Cultural faculty knows iftar timing = local sunset (from the occasions calendar + geolocation), suggests dates and water/soup first, then shareable mains.
2. Scheduled delivery targeted to arrive 10–15 min before sunset.
3. Halal is a **hard constraint** at restaurant level (restaurant-declared certification/practice), not inferred from dish names.
4. Group mode: Aisha shares a link; friends add preferences (no app install required on web).

## 7. Journey F — "Surprise me" (curious eater)

1. Mei taps **Inspire me** → Inspiration dial jumps to 90; Scenic 70.
2. Assistant offers a "flavor passport" step: *"You've loved Sichuan and Mexican heat — ever tried Ethiopian?"* with a short story and a "safe bet" fallback.
3. If she orders, the assistant asks next time how it went and stores an exploration result (loved/liked/not for me).

## 8. Journey G — "Shop extras along the route"

1. After the basket is set: *"Want anything on the way? Drinks, dessert, flowers, diapers?"* (only when the scenic/inspiration dials or the occasion warrant it; express mode never asks unless the user does).
2. Candidates come from shops within the route's detour budget (e.g., ≤ 6 min added), open now.
3. User sets a price cap per item category; courier buys with a capped card; substitutions follow the user's rule (*"same brand any size" / "skip if unavailable" / "message me"*).
4. v1 excludes alcohol, tobacco, cannabis, pharmacy-only medicines.

## 9. Edge journeys (must be designed)

| Situation | Expected behavior |
|---|---|
| A kitchen doesn't answer within 5 min | Escalate channel (SMS → AI call → ops); meanwhile offer a pre-computed substitute dish/restaurant |
| One item unavailable | Offer the restaurant's substitution or our alternative; recompute totals; re-confirm |
| Courier cancels | Auto-failover via dispatch strategy; user notified with new ETA |
| Price changed at the restaurant | If within tolerance (e.g., ≤ 5 %) absorb or notify; otherwise ask the user |
| User edits the basket after payment authorization | Re-authorize if total increases; restaurants re-confirmed only if their leg changed |
| Allergy uncertainty | Never claim "allergen-free"; show "not verified — ask the kitchen" and offer a verified option |
| User mentions self-harm, eating disorder, or medical emergency | Safety protocol: supportive message, resources, no calorie talk; see [AI/09](../03-ai/09-safety-and-health-guardrails.md) |
