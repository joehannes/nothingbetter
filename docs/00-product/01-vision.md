# 01 · Product Vision

> Status: Draft v1 · Owner: Product · Last updated: 2026-10-15

## 1. One-liner

**NothingBetter is a food concierge you talk to first and order from second.** It works out who is eating, why, and in what mood. Then it builds one basket from several restaurants (plus shop extras along the way) and has it delivered together.

## 2. Why this exists

Uber Eats, DoorDash and PedidosYa start from a **catalog**: scroll restaurants, then scroll dishes. Most of the work lands on the eater:

- remembering what everyone at the table likes and cannot eat,
- reconciling a group's wishes ("one wants sushi, one wants pizza, the kid wants plain pasta"),
- thinking about health, the season, the occasion, the budget and the clock,
- inventing variety ("we always order the same three things").

NothingBetter starts from a **conversation and a context**: the time, the weather and season, the calendar event, who is at the table, their health, culture and habits. Only then does it propose food.

> The marketplace is the supply network. The product is the decision.

## 3. The five promises

| # | Promise | What the user feels | What makes it possible |
|---|---|---|---|
| 1 | **It knows the table** | "It remembered Mia's sesame allergy and that Dad hates cilantro." | Avatar system + hard-constraint memory ([AI/05](../03-ai/05-memory-avatars-and-personalization.md)) |
| 2 | **It reads the moment** | Work lunch: three options, one tap. Anniversary: a curated story-led menu. | Adaptive dials: urgency, inspiration, directness, scenic ([AI/04](../03-ai/04-conversation-engine-and-adaptive-dials.md)) |
| 3 | **It advises like a friend with expertise** | Natural-medicine, nutrition, cultural and social insight, never preachy | Council of AI faculties + curated knowledge base ([AI/02](../03-ai/02-ai-faculties-skillsets.md), [AI/06](../03-ai/06-knowledge-base-culture-health-seasons.md)) |
| 4 | **It mixes kitchens freely** | Tacos from one place, Thai curry from another, flowers and sparkling water from the shop on the way | Multi-leg orders + dispatch orchestration ([WF/03](../02-workflows/03-dispatch-multi-pickup-and-shop-extras.md)) |
| 5 | **It confirms before you pay** | "All three kitchens said yes, ready by 7:40." | Restaurant liaison over WhatsApp, SMS, voice and POS ([WF/02](../02-workflows/02-availability-confirmation-whatsapp.md)) |

## 4. Who it is for (launch)

- **The household organizer** — feeds a partner, kids, maybe parents; wants zero-effort weeknights and no allergy accidents.
- **The busy professional** — lunch at the desk, 25 minutes between meetings, wants it fast and decent.
- **The host** — birthday, game night, date night, visiting in-laws; wants it to feel special.
- **The curious eater** — wants to try new cuisines without wasting money on a miss.
- **The health-minded / recovering** — sick day, post-workout, pregnancy, managing a condition; wants supportive food, gently framed.
- **The culturally rooted** — grew up with a cuisine, wants "food like home", observes religious or traditional rules (halal, kosher, fasting periods, vegetarian traditions).

See [03 · Personas & journeys](03-personas-and-journeys.md).

## 5. Principles

1. **Conversation before catalog, but never conversation as a toll booth.** If the user knows what they want, get out of the way.
2. **Ask the one question that changes the answer most.** No 17-step questionnaires (adaptive information gain).
3. **Hard constraints are sacred.** Allergies and diet rules are never "learned away", never guessed, never traded for taste.
4. **Advice, not diagnosis.** Health and natural-medicine insight is framed as guidance with evidence levels and safety checks.
5. **Culture is self-declared.** We never infer anyone's ethnicity, religion or health from their name, face, voice or location.
6. **Remember with consent, forget on request.** Every memory is visible, editable and deletable.
7. **Honest logistics.** If three kitchens mean two couriers and a higher fee, we say so before payment.
8. **Delight scales with the moment.** Playfulness and storytelling are dialed to the occasion, never forced.

## 6. What it is not (v1)

- Not a general-purpose chatbot (it stays on food, hospitality and the meal's social context).
- Not a scraper or reseller of other marketplaces' consumer apps (see [research](02-research-and-partner-landscape.md)).
- Not a medical service; no diagnosis, no treatment claims.
- Not an alcohol seller in v1 (licensing and ID checks deferred to a later phase).
- Not a courier company on day one (we orchestrate couriers; an own/partner pool comes in Phase 3).

## 7. Platforms

- **Tauri 2 app**: macOS, Windows, Linux desktop; iOS and Android mobile — one React codebase.
- **Website**: the same React app on the web (PWA) plus a marketing site.
- **Partner console**: restaurants and shops (web).
- **Ops console**: internal support and dispatch (web).

## 8. Launch geography (recommendation)

North America first, where DoorDash Drive and Uber Direct both operate. Recommended first metro: **Miami-Dade** (bilingual EN/ES, high WhatsApp use among restaurants and customers, dense Latin/Caribbean food scene that matches the founder's background). Second: **Toronto** (multicultural, Canada, adds FR later). This is an open decision — see [risks & open questions](../05-delivery-plan/04-risks-and-open-questions.md).

## 9. Success metrics (north star + guardrails)

| Metric | Definition | Phase 1 target |
|---|---|---|
| **Delighted meals / week** (north star) | Orders where ≥ 1 diner rated a dish "loved" | Grows week over week |
| Proposal acceptance | Sessions where a proposed set is ordered with ≤ 1 edit | ≥ 40 % |
| Time-to-order (express) | First message → payment, urgency ≥ 70 | p50 ≤ 90 s |
| Availability answer time | Check sent → all kitchens answered | p90 ≤ 5 min |
| Order success | Delivered complete and on time (± 10 min of promise) | ≥ 95 % |
| Hard-constraint violations | Allergen/diet errors reaching a diner | 0 (sev-1 incident) |
| Memory trust | Users who open "What I know" and keep memory on | ≥ 85 % keep on |
