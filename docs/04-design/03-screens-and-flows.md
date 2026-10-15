# Design/03 · Screens & Flows

> Status: Draft v1 · Wireframes in text; high-fidelity designs to follow in the design tool of choice.

## 1. Navigation

Mobile: bottom bar — **Home (Host)** · **Orders** · **People** · **Me**. Desktop: left rail with the same, three-pane main area (Conversation | Proposals/Table | Basket/Route).

## 2. Onboarding (≤ 2 minutes)

```text
[1] Welcome — "Tell me who you eat with, I'll handle the rest."  [Get started]
[2] Location in context — "Find kitchens near you?" [Use my location] [Type an address]
[3] Language — auto-detected (EN/ES/FR) with switch
[4] Your table — "Who do you usually eat with?" quick-add chips: Partner · Kids · Parents · Roommates · Just me
     → mini avatar builder per person (name, age band, allergies? diet?)
[5] Taste game — swipe 10 dishes (love / like / nah) + spice slider + "a cuisine that feels like home?" (optional, skippable)
[6] Lenses — Health lens (off/on) · Natural-medicine lens (suggested on) · explained in one line each; consent screen if health data entered
[7] Voice — "Talk to me anytime" mic permission (optional)
[8] Done — first suggestion based on the current moment
```

## 3. Home (Host)

```text
┌──────────────────────────────────────────────┐
│ ContextStrip: [Dinner · 6:10 pm] [Home] [4 at the table ▾] [~$60]
│                                              │
│            ( HostOrb )                       │
│   "Dinner for the four of you tonight?"      │
│   [Yes]  [Just me]  [Change table]           │
│                                              │
│ Dial chips: Just get me food · Inspire me · Keep it healthy │
│ Composer: [ Type or hold to talk…     🎤 ]   │
└──────────────────────────────────────────────┘
```

Express variant: Host line + 3 cards immediately when signals are strong (work lunch).

## 4. Proposals

- **Guided:** horizontal carousel of `MealSetCard`s; tap → expands into **Table view**; secondary "Show everything" opens filterable catalog.
- **Scenic:** vertical story cards with courses (Starter → Mains → Dessert → Extras) and `RouteRibbon`.
- **Express:** three `ProposalCard`s with big *Order* buttons.

Each card: image, dish/merchant, per-seat badges, price, ETA window, couriers count, "Why this?" link.

## 5. Table view (signature)

```text
            ( Rosa )
       🍲 Low-sodium sopa
 ( Ana )                 ( Carlos )
 🥗 Quinoa bowl           🍖 Ropa vieja
       ( Mia ✓sesame-safe )  ( Leo ♥ )
         🍝 Kids pasta       🍕 Margherita
 Shared center: 🥟 Empanadas x6
 Total $72.40 · 2 kitchens · 1 courier · 7:20–7:30
 [Check & hold]
```

Interactions: drag dish between seats; tap seat → alternatives for that person; long-press dish → details; add guest seat.

## 6. Basket & Check & hold

- Grouped by **stop** (kitchen/shop) with thermal icons and per-leg totals.
- Extras section with caps and substitution rules.
- `PriceBreakdown` (estimate) → **Check & hold** primary button.
- After tapping: `KitchenCheckChips` animate per merchant: *Sent via WhatsApp → Read → ✅ Ready by 7:40*. Changes surface inline with one-tap backups.
- When all ✅ → **Pay $72.40** (Apple Pay/Google Pay/card) with hold countdown ("Kitchens holding for 9:12").

## 7. Tracking

- Map with route and stops; stop chips update (🍰 picked up · 💐 bought · 🍽️ cooking · 🏠 7:28).
- Synchronized legs: "2 couriers · arriving within 8 min of each other".
- Delivered: photo proof → feedback prompt (later, not intrusive).

## 8. Feedback

Per seat: 😍 / 🙂 / 😐 / 👎 for each dish (one tap each), optional "what made it great?" chips (flavor, portion, temperature, presentation). Celebrations: "Save this as a memory?" (with photo optional).

## 9. People

- Household and circles; avatar cards with constraints badges.
- Add person (quick) / invite to claim.
- Per-avatar "What I know" (memory) with edit/delete.

## 10. Me / Settings

Default dials per moment (with plain-language labels), lenses, voice (mode, voice style, speed), notifications & proactive suggestions, privacy (incognito default, data export/delete), payment methods, places, language.

## 11. Group link (web, no install)

Invitee sees: host's occasion, their seat, choices filtered by shared constraints, can add own constraints privately → host sees a merged Table view.

## 12. Empty, loading, error states

| State | Pattern |
|---|---|
| Proposals loading | Skeleton cards + Host thinking swirl + one progress line ("Checking 14 kitchens near you…") |
| No feasible options | Host explains the binding constraint and offers 2 relaxations |
| Location denied | Address search + saved places; no nagging |
| Mic denied | Explain how to enable per OS; continue with text |
| Offline | Last session read-only; queue text until online |
| Kitchen didn't answer | Backup offer inline; never a dead end |
