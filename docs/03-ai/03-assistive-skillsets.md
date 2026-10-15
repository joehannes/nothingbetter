# AI/03 · Assistive Skillsets (what the assistant does for people)

> Status: Draft v1 · Faculties (AI/02) are *internal experts*; assistive skills are *user-facing capabilities* the Host offers. Each skill lists its trigger, the experience, and success criteria.

## 1. Human desirables for a food ordering platform

What people actually want, ranked by how often it decides whether they come back:

| Desirable | What it means in practice | How NothingBetter delivers it |
|---|---|---|
| **Speed when it matters** | Order in under a minute when hungry/busy | Express lane, one-tap reorder, POS-instant availability |
| **No mistakes** | Right items, allergies respected, nothing missing | Hard constraints, restaurant-verified allergen labels, per-leg checklists, photo proof |
| **Hot food, on time** | Arrives at promised time, good temperature | Kitchen timing, thermal sequencing, honest ETAs, travel-quality warnings |
| **Fair, transparent cost** | No surprise fees | All fees shown per leg before checks; price-change tolerance rules |
| **Less decision fatigue** | "Just tell me what's good" | 3 tailored options, Table view, "decide for me" |
| **Everyone happy** | Kids, partner, guests, grandma | Seats, group sets, compatible menus |
| **Variety without risk** | New things that land | Flavor bridges, "safe bet + wildcard" |
| **Feeling cared for** | Sick day, celebration, rough week | Care mode, occasion playbooks, warm-but-not-cheesy tone |
| **Control & privacy** | Know and edit what's remembered | "What I know about you", incognito session |
| **Easy fixes** | Quick refunds/redelivery when wrong | Self-serve issues with instant partial refunds within limits |
| **Accessibility** | Voice, large text, screen readers, low literacy | Voice-first parity, plain-language mode |

## 2. Assistive skill catalog

| # | Skill | Trigger | Experience | Success criteria |
|---|---|---|---|---|
| A1 | **Decide for me** | "Surprise me", "you choose", or no preference after 1 question | One confident proposal + one alternative, with a short why | Accepted ≥ 50 % |
| A2 | **Set the table** | Session start with household | "Dinner for the four of you?" seats pre-filled; tap to add/remove people | Seats correct in ≥ 90 % without edits |
| A3 | **Group consensus** | ≥ 3 seats or shared link | Crowd-pleaser + personal picks; optional quick poll to guests (web link, no install) | Group sets accepted with ≤ 2 edits |
| A4 | **Smart reorder** | Repeated context (Friday pizza) | "Same as last Friday? Or a twist?" with one-tap | Reorder in ≤ 2 taps |
| A5 | **Health-aware swap** | Health lens on | Inline "lighter/protein-forward/lower-sodium version" chips on cards | Swap used, satisfaction ≥ original |
| A6 | **Care mode** | Sick, sad, stressed, recovering | Short, gentle tone; comfort/recovery foods; optional shop extras (tea, honey, soup) | No upsell; positive feedback |
| A7 | **Occasion concierge** | Birthday, anniversary, holiday, game night | Course plan, timing, extras (flowers, candles, cake), scheduled delivery | On-time within window ≥ 95 % |
| A8 | **Food like home** | Heritage declared; "homesick", "like my mom's" | Matching dishes and kitchens; regional variants explained | "Loved" rating rate |
| A9 | **Flavor passport** | Curious eater, inspiration ≥ 60 | Bridge from loved flavors to a new cuisine; story card; post-meal "how was it?" | Exploration "liked+" ≥ 60 % |
| A10 | **Explain this dish** | Tap on any dish | Ingredients (estimated vs declared), spice level, origin story, nutrition estimate, traditional view | — |
| A11 | **Menu translator** | Non-native language or unfamiliar dish names | Dish names explained in user's language | — |
| A12 | **Budget guard** | Budget set or detected | Totals per person; cheaper equivalent sets; fee-saving mode (fewer legs) | Under budget ≥ 95 % |
| A13 | **Leftovers & tomorrow** | Family/group or meal-prep users | Suggest portion sizes that cover tomorrow's lunch; reheating notes | — |
| A14 | **Send a meal** | "Send food to my sister", condolence, new baby | Order for another address with gift note; respects recipient's known constraints if they share them | Recipient delight (optional feedback) |
| A15 | **Plan ahead** | Calendar event in future, holidays | Pre-plan and schedule, reminders, re-confirm availability day-of | — |
| A16 | **Remind & anticipate** (opt-in) | Birthdays, rituals, observances | Gentle, capped nudges ("Rosa visits Sunday — book her low-sodium favorites?") | ≤ 2 proactive nudges/week; opt-out honored |
| A17 | **Taste onboarding game** | First run | Swipe dishes (love/like/nah), spice slider, "cuisine that feels like home" | ≤ 90 s to first useful profile |
| A18 | **Shop-along extras** | Basket set, scenic/occasion | Drinks, dessert, flowers, essentials from shops on the route with price caps & substitution rules | Extras delivered as requested ≥ 95 % |
| A19 | **Fix it** | Missing/wrong/late | 2-tap issue report, instant refund/credit within policy, redelivery option | Resolution ≤ 2 min for standard cases |
| A20 | **Why this?** | Tap on any proposal | Transparent reasons (seat fit, health, timing, price), what's remembered | — |

## 3. Proactivity policy

- Off by default except order-related notifications. Users opt in to **rituals & reminders** per avatar or occasion.
- Frequency caps: ≤ 2 proactive suggestions per week; none during quiet hours; none during declared fasting periods.
- Every proactive message carries "Stop these" and "Why am I seeing this?".
- Never proactive about health conditions ("Your blood pressure…") — health is used only when the user starts a session.

## 4. Tone principles per skill

| Context | Tone | Example |
|---|---|---|
| Express / work | Crisp, zero filler | "Three picks, here by 12:35." |
| Family weeknight | Friendly, practical | "Leo's favorite is open tonight — want it for him?" |
| Celebration | Warm, a little playful, evocative | "Five years deserves a little theater. Here's a three-course journey…" |
| Care | Soft, short, no exclamation marks | "Sorry you're not feeling well. A few gentle options:" |
| Cultural | Respectful, curious, never exoticizing | "Iftar starts at 7:12 pm — dates and harira first, then something to share?" |
| Discovery | Enthusiastic, specific | "You love smoky heat — Oaxacan tlayudas are your next stop." |

Copy guidelines in [design/04](../04-design/04-voice-ux-and-conversation-design.md).
