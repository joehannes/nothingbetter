# AI/04 · Conversation Engine & Adaptive Dials

> Status: Draft v1 · Live adaptation of urgency, inspiration, directness and scenicness of the ordering journey.

## 1. The four dials (+ two secondary)

| Dial | 0 means | 100 means | Main effects |
|---|---|---|---|
| **Urgency** | Plenty of time, no rush | "I need food now / I have 10 minutes" | Number of questions, number of options, LLM effort, auto-skip of extras, checkout shortcuts, ETA weight |
| **Inspiration** | Stick to known favorites | Surprise me, novelty welcome | Novelty bonus in scoring, Inspiration Curator activation, stories, wildcard dish |
| **Directness** | Warm, conversational, host-like | Terse, factual, no pleasantries | Message length, greeting/closing, emoji, explanations depth |
| **Scenic** | Express lane: shortest path to order | Scenic route: immersive journey, visuals, route stops, courses | UI lane, imagery, Table view, extras offers, course structure |
| *Health emphasis* (secondary) | Health not a factor today | Health is a priority | Nutrition/Natural-medicine weights, swap chips |
| *Budget sensitivity* (secondary) | Price no concern | Tight budget | Value Analyst weight, fewer legs, portion strategy |

**Lane** is derived: `express` if scenic < 35 or urgency ≥ 75; `scenic` if scenic ≥ 65 and urgency < 50; else `guided`.

## 2. Signals → dial updates

| Signal | Source | Effect (typical) |
|---|---|---|
| Time to next calendar event < 45 min | Calendar (opt-in) | urgency +25 |
| Workday 11:00–14:00 on desktop | Device + time | urgency +10, directness +15, scenic −15 |
| Words: "quick", "asap", "starving", "just" | Message | urgency +20, directness +10 |
| Words: "special", "celebrate", "anniversary", "birthday", "surprise" | Message/calendar | scenic +30, inspiration +20, urgency −10 |
| Words: "something new", "bored of", "surprise me" | Message | inspiration +30 |
| Words: "sick", "tired", "rough day" | Message | directness +10, scenic −10, health +25 (care mode) |
| Short replies (< 4 words), fast taps | Behavior | urgency +5 per turn (cap +15), directness +5 |
| Long, descriptive messages, questions back | Behavior | scenic +5, directness −5 |
| Voice speech rate > user baseline × 1.2 | Voice | urgency +10 |
| Explicit chips: **Just get me food** / **Take me on a journey** / **Inspire me** / **Keep it healthy** | UI | Set dial to preset (override) |
| Budget mentioned | Message | budget sensitivity +30 |
| Late night (after 22:00) | Time | directness +10, scenic −10 |
| Weekend evening with ≥ 2 seats | Context | scenic +10 |

Signals are defined in `knowledge/rulesets/dials/signals.yaml` and unit-tested.

## 3. Dial engine algorithm

```text
prior      = user's learned defaults for this context cluster (meal slot × weekday/weekend × device × table size)
ruleset    = occasion/situation preset (e.g., work-lunch: U70 I15 D85 S10)
live       = Σ signal deltas with exponential decay (half-life 3 turns)
model_nudge= Host's set_dials() suggestion, bounded ±15/turn
override   = explicit user chip (sticky for the session)

value = clamp( override ?? (0.35·prior + 0.35·ruleset + live + model_nudge), 0, 100 )
hysteresis: lane changes only when the threshold is crossed by ≥ 8 points
learning: at session end, prior ← EMA(prior, final dials, α=0.2) for that context cluster
```

Implementation: pure function in `packages/domain/dials.ts`; explains itself (`reasons[]`) for the UI ("Kept it quick — you have a meeting at 12:45").

## 4. Effects table (what the dials change)

| Behavior | Express (U≥75 or S<35) | Guided | Scenic (S≥65, U<50) |
|---|---|---|---|
| Max clarifying questions before proposals | 0–1 | ≤ 2 | ≤ 3 (playful) |
| Options shown | 3 cards | 3–4 sets + Table view | 3 curated journeys with courses |
| Message length (directness high → low) | ≤ 25 words | ≤ 60 words | ≤ 120 words incl. story |
| Stories & cultural notes | Off | On tap | Inline (1–2 sentences) |
| Imagery | Thumbnails | Cards | Large hero imagery, route map |
| Extras offer | Never unless asked | If relevant | Yes, curated (flowers, dessert, drinks) |
| Host effort | `low` | `medium` | `medium` (`high` for groups ≥ 6) |
| Voice TTS rate | 1.1× | 1.0× | 0.95× with warmer voice style |
| Checkout | One-tap saved method | Standard | Standard + schedule & gift note |
| Feedback ask | Next session, 1 tap | After delivery | After delivery + "story" prompt |

## 5. Conversation policy (Host)

1. **Open with context, not a question list.** "Lunch · 40 min until your 12:45" chips + mic.
2. **Ask the single best question** (highest expected information gain over current candidate sets). If the expected gain of asking is lower than the cost of asking (urgency-weighted), propose instead.
3. **Propose early, refine by reaction.** Proposals are themselves questions ("Closer to A or B?").
4. **Always offer escape hatches:** "Just order my usual", "Show me everything" (catalog view), "Talk to a human".
5. **Never repeat questions already answered** (brief + memory).
6. **Honor hard constraints silently and visibly** (badges), not with lectures.
7. **Summarize before commitment:** items per seat, merchants, total, ETA, fees, dispatch mode.
8. **Language mirroring:** reply in the user's language and register; support code-switching (EN/ES).

### Question selection (expected information gain)

```text
candidates = current top-K meal sets (K≈20) with scores
for each possible question q in the question bank (+ faculty-suggested questions):
   for each answer a in q.answers with P(a) from priors:
       H_after += P(a) · entropy(score distribution of candidates | a)
   gain(q) = H_now − H_after
ask argmax gain(q) if gain > τ(urgency)  else propose
```

Question bank lives in `knowledge/templates/questions/*.yaml` with answer chips and how each answer maps to filters/weights.

## 6. Prompt mechanics for live adaptation

- The Host system prompt is **frozen** (cached). Dials and situation arrive as a **mid-conversation system message** after each user message:

```text
[live-context v3]
lane=express urgency=82 inspiration=15 directness=88 scenic=10 health=40 budget=55
time=2026-10-20T12:05-04:00 slot=lunch next_event_in=40m
table=[av_priya(self)] place=Office (Bay St)
style: ≤25 words, no greeting, 3 options max, no stories, no extras.
```

- Effort per turn follows urgency (beta per-message effort, else top-level `effort` on the next request).
- Model-proposed dial changes go through `set_dials` (bounded) so the UI and the engine stay in sync.

## 7. Example: same request, three lanes

User: *"Something with chicken."*

- **Express (U85/D90):** "Three picks, all here by 12:40: 1) Chicken shawarma bowl $14 2) Thai basil chicken $15 3) Grilled chicken burrito $13. Tap to order."
- **Guided (U50/D55):** "Got it. Spicy or mild today? Meanwhile, here are three that fit your usual: …"
- **Scenic (U25/S80/I70):** "Chicken can travel the world tonight — a smoky Peruvian pollo a la brasa, a Hyderabadi biryani with saffron, or a Korean double-fried crunch with a cooling pickled radish. Want me to plan a little tasting journey for two?"

## 8. Metrics

- Dial accuracy: share of sessions where the user overrides the lane (target ≤ 15 %).
- Time-to-order by lane; proposal acceptance by lane; satisfaction by lane.
- Over-politeness complaints / "too long" feedback (target ↓).
