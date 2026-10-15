# AI/05 · Avatars, Memory & Personalization

> Status: Draft v1 · Data design in [04 Data model](../01-architecture/04-data-model.md#2-people--memory) · Privacy in [06](../01-architecture/06-security-privacy-compliance.md)

## 1. Avatar system

An **avatar** is a person the user orders for. The account holder is always avatar #1 (`relationship=self`).

```text
User (account holder, 18+)
 └─ Household "Home"            └─ Circle "Office lunch crew"   └─ Circle "Book club"
     ├─ Carlos (self)               ├─ Carlos (self)                 ├─ …
     ├─ Ana (partner)               ├─ Dev (coworker, vegan)
     ├─ Mia (child, 8) ⚠ sesame     └─ Jo (coworker)
     ├─ Leo (child, 13)
     └─ Rosa (grandparent) — low sodium, visits Sundays
```

### Avatar card (UI + data)

| Field | Type | Notes |
|---|---|---|
| Name / nickname | text | "Mom", "Mia" |
| Relationship | enum | self, partner, child, parent, grandparent, sibling, friend, coworker, guest, other |
| Age band | enum | infant, child, teen, adult, older adult (no birthdate needed; optional birthday for reminders) |
| Look | illustrated avatar builder | No photos of minors; optional photo for adults |
| **Hard constraints** | list | Allergies (with severity), intolerances, diet (vegan, vegetarian, pescatarian, keto…), religious (halal, kosher/kosher-style, no pork, no beef, Jain…), medical (low sodium, low sugar, soft foods), "never" items |
| Health context (opt-in) | encrypted | Goals (energy, muscle, weight management, recovery), conditions, pregnancy, medications (optional, only for interaction checks) |
| Heritage & observances (opt-in) | list | Self-declared; relation: grew up with / family heritage / lives by / curious about |
| Languages | list | For dish names and voice |
| Taste profile | learned | Cuisines, dishes, flavors, textures, spice tolerance, adventurousness, portion size |
| Rituals | list | "Pizza Fridays", "Sunday family lunch with Rosa" |
| Important dates | list | Birthday, anniversary, name day (opt-in reminders) |
| Notes | text | Free notes ("hates soggy fries") |
| Ownership | system | Owner, linked user (if claimed), visibility in shared group links |

### Creating avatars without friction

- **Quick add from conversation:** "My mother-in-law is coming, she's diabetic" → Memory Curator proposes *"Add 'Mother-in-law' as a guest with 'low sugar' (medical)?"* → one-tap confirm.
- **Taste game per avatar:** 10 swipes, can be done by the person themselves via an invite link.
- **Guest seats:** one-off seats with constraints that are not saved unless the user chooses.
- **Claiming:** adults can be invited to claim their avatar → they gain control; the creator keeps only what the claimant shares.

## 2. Memory model

| Layer | What | Write rule | Read rule | Decay |
|---|---|---|---|---|
| **M1 Hard facts** | Allergies, diet & religious rules, medical restrictions | Only explicit user confirmation | Always loaded for present seats | Never (until edited) |
| **M2 Preferences** | Taste likes/dislikes with confidence, context scope | Auto from ratings/orders; confirmations for strong statements | Top-K relevant to brief | Half-life 180 d default; stated prefs 365 d |
| **M3 Episodes** | "Loved the Peruvian tart on their anniversary" | Auto after orders/feedback; summarized | Similarity to brief (≤ 5) | Summarize after 12 months |
| **M4 Rituals & dates** | Recurring patterns, important dates | Proposed after ≥ 3 occurrences or stated; user confirms | When slot/date matches | Retire after 3 misses |
| **M5 Situational** | "Tonight I want something light" | Session only | Session only | Ends with session |
| **M6 Dial priors** | Default dials per context cluster | Auto (EMA) | Session start | Continuous |

Key rule from the earlier investigation: **situational wishes never overwrite stable preferences.**

## 3. Learning signals

| Signal | Strength | Applies to |
|---|---|---|
| Dish rating per seat (loved/liked/meh/no) | Strong | Seat's dish/cuisine/flavor prefs |
| Reorder of same dish | Medium | Dish pref, ritual detection |
| Explicit statement ("Leo hates mushrooms") | Strong (confirm) | Preference or hard constraint (if allergy/"never") |
| Proposal skipped / swapped away | Weak | Negative evidence, context-scoped |
| Leftover feedback ("too much") | Medium | Portion size |
| Exploration result | Medium | Adventurousness, cuisine bridges |
| Dial overrides | Medium | Dial priors |

Update rule (preferences): Bayesian-style confidence update with evidence count; polarity = weighted mean of evidence; decay applied at read time.

## 4. Memory Curator flow

```text
turn/order/feedback ─▶ extract candidates (structured call) ─▶ classify layer + sensitivity
      ─▶ dedupe/merge vs existing (embedding + key match)
      ─▶ M1 or sensitive? ── yes ─▶ memory.proposed event → user confirms in UI/voice
                         └─ no  ─▶ write with provenance (silent), visible in "What I know"
```

Voice confirmation example: *"Want me to remember that Mia is allergic to sesame — severe?"* → "Yes" → stored with `source=user_stated`, `confirmed_at`.

## 5. Transparency & control ("What I know about you")

- Per avatar: hard constraints, preferences (with confidence bars and "because…"), memories (with "from your order on Oct 3"), rituals, dial defaults.
- Actions: edit, delete, "never use this", export, pause learning, incognito session ("don't remember this session").
- Household view shows only what each avatar's owner permits.

## 6. Group links & privacy between people

- Shared group order links show guests only **what they need**: their own seat, dish choices, and shared constraints that affect the shared menu (e.g., "nut-free table").
- One person's health data or heritage is never revealed to others; constraints affecting shared dishes are phrased neutrally ("the table is sesame-free").

## 7. Personalization in ranking

`seatFit(dish, avatar, context)` components:

| Component | Source | Weight driver |
|---|---|---|
| Taste match | M2 vs dish flavor/texture/cuisine embedding | Base |
| Proven favorite | M2/M3 ratings | Lower when inspiration high |
| Novelty bonus | Distance from history, bridge score | Inspiration dial |
| Health fit | F3/F4 boosts | Health emphasis |
| Cultural fit | F5 boosts (only if heritage declared or occasion) | Occasion |
| Social fit | F7 (shareability, kid-friendly) | Table size |
| Price fit | F10 | Budget sensitivity |
| Travel quality | Dish travel score × route time | Always |
| Merchant reliability | Answer rate, accuracy, lateness | Always |

Weights are versioned in rulesets and tuned offline with logged outcomes (counterfactual evaluation before rollout).
