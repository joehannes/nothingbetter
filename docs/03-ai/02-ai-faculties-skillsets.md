# AI/02 · AI Faculties & Skillsets

> Status: Draft v1 · Each faculty = a versioned system prompt + knowledge pack + output schema + eval set. Prompts live in `knowledge/templates/prompts/faculties/*.md`.

## 1. Faculty roster

| # | Faculty | Mission (one line) | Activation | Required? |
|---|---|---|---|---|
| F0 | **Safety Sentinel** | Keep every diner safe: allergens, diet rules, medical red flags, interactions, minors, crisis signals | Always | Yes |
| F1 | **Context Reader** | Turn signals (time, calendar, weather, place, words) into a precise Situation Brief | Every session start + on signals | Yes (mostly code) |
| F2 | **Palate Profiler** | Know what each seat loves, tolerates, avoids, and how adventurous they are today | ≥ 1 avatar with history | — |
| F3 | **Nutrition Scientist** | Fit meals to goals, conditions and life stage with evidence-based nutrition | Health lens on, goals set, or user asks | — |
| F4 | **Natural Medicine Advisor** *(preferred lens)* | Bring traditional food wisdom (seasonal, warming/cooling, digestion, herbs & spices) with evidence grades and safety | Natural-medicine lens on (opt-in, default suggested at onboarding) | — |
| F5 | **Cultural Anthropologist** | Honor heritage, religious rules, festivals and "food like home"; open doors to new cuisines respectfully | Any declared heritage/observance, cultural occasion, or curiosity | — |
| F6 | **Occasion Planner** | Shape the meal to the moment: work lunch, date night, birthday, game night, grief, new baby, holidays | Occasion detected or declared | — |
| F7 | **Social Intelligence** | Read the table: group dynamics, guests, kids, hosts' goals, gifts and gestures, etiquette | ≥ 2 seats or social occasion | — |
| F8 | **Body & Activity Coach** | Match energy needs and timing: training days, long shifts, late nights, sleep, hydration | Activity signals or goals | — |
| F9 | **Inspiration Curator** | Add surprise, story and discovery at the right dose | Inspiration dial ≥ 40 or "Inspire me" | — |
| F10 | **Value Analyst** | Maximize value within budget: portions, sharing, fees, deals, leftovers | Budget set or budget sensitivity ≥ 50 | — |
| F11 | **Logistics Planner** | Make it arrive right: prep times, travel quality, legs vs. runs, route, shop extras, timing | Always before proposals | Yes |
| F12 | **Memory Curator** | Decide what's worth remembering, with consent and provenance | After turns/orders (async) | — |

Ops agents (not consulted by the council): **Restaurant Liaison** ([WF/02](../02-workflows/02-availability-confirmation-whatsapp.md)), **Menu Parser & Enricher** ([AI/07](07-background-ingestion-and-parsing.md)), **Support Agent** ([WF/07](../02-workflows/07-support-refunds-and-exceptions.md)).

## 2. Faculty specifications

### F0 · Safety Sentinel
- **Inputs:** seats' hard constraints, health opt-ins, medications (if provided), minors, messages, candidate dishes (later pass).
- **Knowledge:** allergen ontology (US 9 major allergens incl. sesame; Canada priority allergens incl. mustard, sulphites), cross-contact risk by cuisine/technique, food–drug & herb–drug interactions (e.g., grapefruit with certain statins/calcium-channel blockers; vitamin-K–rich foods with warfarin; St John's wort; licorice and blood pressure), pregnancy food safety (raw fish, unpasteurized cheese, high-mercury fish, alcohol), infant rules (no honey < 12 months), choking hazards for small children, eating-disorder-sensitive language, crisis resources.
- **Skills:**
  1. Constraint compilation → SQL-ready filters per seat.
  2. Allergen risk estimation on dishes with unverified data (blocks for anaphylaxis-level allergies).
  3. Interaction screening (only when medications are provided; output cautions, never dosing advice).
  4. Red-flag detection in messages (medical emergency, self-harm, disordered eating) → `safety.notice` + resources, and stop calorie talk.
  5. Age-appropriateness (spice for toddlers, caffeine for kids).
- **Output emphasis:** `hardConstraints`, `cautions(block|warn)`.
- **Guardrails:** cannot be overridden by any other faculty or dial; can only be overridden by the user for non-allergy *soft* cautions with an explicit tap.

### F1 · Context Reader
- **Inputs:** device time & TZ, place, calendar (opt-in: Google/Microsoft/Apple via OS), weather API, session words, recent orders, device type, typing/voice cadence.
- **Skills:** meal-slot detection (breakfast/brunch/lunch/afternoon/dinner/late night), time-pressure estimation (next event), occasion detection ("birthday", "game", "date", "Ramadan"), mood cues ("rough day", "celebrating"), who's present (household defaults + words: "the kids are with me"), weather → comfort/cooling foods, season & local produce.
- **Output:** fields of the Situation Brief + signals for the dial engine (see [AI/04](04-conversation-engine-and-adaptive-dials.md)).
- **Guardrails:** no inference of sensitive attributes; calendar titles used only in-session, not stored unless the user saves the occasion.

### F2 · Palate Profiler
- **Inputs:** avatar preferences, ratings, order history, explicit dislikes, exploration results.
- **Skills:** taste-vector matching (flavor, texture, spice, richness), "comfort set" detection (top reliable dishes), boredom detection (same dish N times → suggest a sibling dish), context-scoped tastes (lunch vs celebration), adventurousness today (from words + dials).
- **Output:** per-seat boosts and avoid cautions.

### F3 · Nutrition Scientist
- **Inputs:** goals, conditions (opt-in), life stage, activity, dish nutrition estimates with confidence.
- **Knowledge:** Dietary Guidelines for Americans and Canada's Food Guide principles; condition-specific patterns (e.g., DASH-style lower sodium for hypertension, carbohydrate awareness for diabetes, low-FODMAP awareness for IBS, iron/folate in pregnancy), protein needs for training, fiber, satiety.
- **Skills:**
  1. Meal-level macro and sodium estimation with confidence bands.
  2. "Better swap" suggestions (side salad instead of fries, broth on the side, grilled vs fried).
  3. Balanced group meals (one shared vegetable-forward dish).
  4. Gentle framing generator: *profile → why it might matter → alternative*.
- **Guardrails:** no diagnosis, no treatment claims, no calorie talk if disordered-eating flag; numbers shown as estimates; evidence level shown on tap.

### F4 · Natural Medicine Advisor (preferred lens)
- **Inputs:** season, weather, symptoms the user volunteers (e.g., "getting over a cold", "bloated", "can't sleep"), constitution preferences if the user opts into a tradition (e.g., TCM or Ayurveda), medications for interaction screening (via F0).
- **Knowledge:** traditional systems mapped onto dishes/ingredients: **Traditional Chinese Medicine** food natures (warming/cooling/neutral) and flavors; **Ayurveda** rasa/virya/dosha effects; **Western herbal and folk traditions** (ginger, garlic, chamomile, fennel, peppermint, turmeric, bone broth, fermented foods); **seasonal eating** by North American region; **culinary herbs & spices** with evidence notes (NCCIH, MSK About Herbs, NIH ODS as references).
- **Skills:**
  1. Seasonal balance (warming soups and spices in cold months; cooling, hydrating foods in heat).
  2. Recovery & comfort meals (congee, broths, khichdi, sopa de pollo, miso).
  3. Digestion-friendly choices (lighter evening meals, gentle spices, fermented sides).
  4. Herbal tea / tonic add-ons from shops on the route (Phase 3).
  5. Evidence-graded micro-explanations: *"Ginger has a long tradition for nausea; clinical studies support modest benefit (B)."*
- **Guardrails:** traditional framing language ("traditionally used", "many people find"); every claim carries an evidence level A–D; never replace medical care; interaction screening via F0 (e.g., turmeric/ginger with anticoagulants, licorice with hypertension); pregnancy and children → conservative mode.

### F5 · Cultural Anthropologist
- **Inputs:** self-declared heritage and observances per seat, occasion calendar, locale, curiosity.
- **Knowledge:** cultural packs ([AI/06](06-knowledge-base-culture-health-seasons.md)): cuisines, signature and comfort dishes, eating styles (family-style, individual plates, sharing etiquette), religious food rules (halal, kosher/kosher-style, Hindu/Jain vegetarianism, Lent/fasting, Buddhist vegetarian days), festival foods (Lunar New Year, Diwali, Eid, Passover, Thanksgiving US/Canada, Día de Muertos, Nochebuena, Juneteenth, Lunar festivals), diaspora variations in North America.
- **Skills:** "food like home" matching (canonical dish families, regional variants), respectful discovery bridges ("If you love mole, try Oaxacan tlayudas or Ethiopian doro wat"), festival timing (iftar at sunset, Shabbat before Friday sundown), mixed-heritage tables (find shared ground), language-aware dish naming.
- **Guardrails:** never assume heritage; avoid stereotypes ("since you're X you'll want Y"); present traditions as options, not prescriptions; certification claims only from merchant declarations.

### F6 · Occasion Planner
- **Inputs:** occasion type (detected/declared), calendar event, guests, budget, time.
- **Skills:** occasion playbooks (templates in `knowledge/rulesets/occasions/*`): work lunch, team lunch, date night, anniversary, birthday (kids/adult), game night, movie night, holiday feast, potluck supplement, condolence meal (send food to someone grieving: gentle, practical, freezer-friendly), new parents (one-handed foods), exam/crunch week, hangover/recovery brunch, picnic. Each playbook defines course structure, shareability, presentation, timing, extras (candles, flowers, balloons), tone.
- **Output:** course plan + ideas + extras suggestions + tone hint.

### F7 · Social Intelligence
- **Inputs:** seats, relationships, group size, occasion, messages ("my in-laws are picky", "first date").
- **Skills:**
  1. Group consensus building (one crowd-pleaser + personal picks; minimize "everyone compromises").
  2. Host support (what impresses without stress; serving style, timing so food arrives before guests).
  3. Kid-friendly strategy (familiar + one adventurous bite; mild spice; separate sauces).
  4. Elders' comfort (soft textures, lower sodium, familiar flavors).
  5. Etiquette & gestures (bring dessert, dietary inclusivity for guests, send a meal as a gift with a note).
  6. Conflict avoidance (no pork if one guest avoids it → choose a fully compatible menu when the group is small).
- **Guardrails:** no gossip or judgments about absent people; respect privacy of other avatars (don't reveal one person's health data to others in shared group links).

### F8 · Body & Activity Coach
- **Skills:** pre/post-training meals (protein + carbs timing), long-shift fuel, late-night lighter options, sleep-friendly evening choices (less caffeine, lighter meals), hydration add-ons, travel/jet-lag meals.
- **Guardrails:** no body shaming; no weight talk unless the user set a weight-management goal; disordered-eating flag disables numbers.

### F9 · Inspiration Curator
- **Skills:** novelty dosing (explore/exploit by dial), "flavor passport" bridges from loved flavors to new cuisines, micro-stories (origin, a chef's twist, the season), themed nights ("Street food of Mexico City", "A Lisbon evening"), seasonal specials, "one wildcard dish for the table".
- **Guardrails:** stories must be factual and sourced from knowledge entries or menu text; no invented chef biographies.

### F10 · Value Analyst
- **Skills:** cost per person, family-size vs individual portions, shareable mains, leftover planning ("tomorrow's lunch"), fee-aware merchant selection (one fewer leg saves $X), budget guardrail messaging.

### F11 · Logistics Planner
- **Inputs:** merchant locations, prep times, live load (if known), DaaS quotes, own-fleet availability, thermal classes, travel scores, route, shops along route, weather/traffic.
- **Skills:**
  1. Feasibility (open now, deliver by T, within zones).
  2. Mode selection: single / synchronized legs / concierge run with fee & ETA comparison.
  3. Pickup sequencing by thermal class (ambient → cold → hot last; frozen at the end or insulated).
  4. Kitchen timing (each merchant's `ready_by`).
  5. Detour search for shop extras (route corridor, opening hours, category).
  6. Travel-quality warnings ("Fries over 20 min travel get soft — want a crispier alternative?").
- **Output:** `LogisticsPlan` per meal set (mode, legs, ETAs, fees, warnings).

### F12 · Memory Curator
- **Skills:** extract candidate facts from conversation and outcomes; classify (hard fact, preference, episode, ritual, date); dedupe/merge with existing memories; decide ask-to-confirm vs silent soft update; set decay; write provenance.
- **Guardrails:** hard constraints and sensitive data always require explicit confirmation; never store third-party sensitive info beyond safety needs; respects "don't remember this" and incognito sessions.

## 3. Skill → faculty matrix (who contributes what)

| Skill area | F0 | F2 | F3 | F4 | F5 | F6 | F7 | F8 | F9 | F10 | F11 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Allergens & diet rules | ● | | ○ | ○ | ○ | | | | | | |
| Taste fit | | ● | | | ○ | | ○ | | ○ | | |
| Health & goals | ○ | | ● | ● | | | | ● | | | |
| Seasonal/traditional | | | ○ | ● | ● | | | | ○ | | |
| Heritage & religion | ○ | | | | ● | ○ | ○ | | | | |
| Occasion structure | | | | | ○ | ● | ● | | ○ | ○ | ○ |
| Group harmony | | ○ | | | ○ | ○ | ● | | | ○ | |
| Novelty & story | | ○ | | ○ | ○ | ○ | | | ● | | |
| Budget | | | | | | ○ | | | | ● | ○ |
| Timing & delivery | | | | | ○ | ○ | ○ | ○ | | ○ | ● |

● primary · ○ contributes

## 4. Adding a new faculty (checklist)

1. Write `knowledge/templates/prompts/faculties/<id>.md` (front matter: id, version, effort, activation, output schema ref).
2. Add knowledge pack sources with licenses and reviewer.
3. Extend `FacultyId` and ruleset activation conditions.
4. Add ≥ 30 eval cases incl. adversarial ones ([AI/11](11-evaluation-and-quality.md)).
5. Shadow-run (outputs logged, not used) for one week; then enable via flag.
