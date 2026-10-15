# Design/01 · Design Principles & Brand

> Status: Draft v1 · "Beautiful, playful, intuitive — a warm host, not a vending machine."

## 1. Brand idea: *the table*

Everything in NothingBetter revolves around **a table with people at it**. Food apps show shelves of restaurants; we show *your* table — who's sitting there tonight, what each person will love, and how it all arrives together.

- **Personality:** a warm, witty friend who cooks and knows every kitchen in town. Generous, curious, never preachy, quick when you're busy.
- **Voice:** plain words, specific details, light humor in celebrations, softness in care moments, crispness at work.
- **Promise line (working):** *"Nothing better than the right food, for the right people, right now."*

## 2. Design principles

| Principle | Means | Anti-pattern we avoid |
|---|---|---|
| **People before products** | Seats and faces first, dishes second | Endless restaurant lists |
| **One clear next step** | Every screen has a single primary action | Five competing CTAs |
| **Dial the delight** | Playfulness scales with the scenic dial and occasion | Confetti on a work lunch |
| **Show, then tell** | Cards, imagery and the Table view carry meaning; text explains on demand | Walls of AI text |
| **Honest by default** | Fees, couriers, timing and uncertainty visible before paying | Surprise fees, fake precision |
| **Calm confidence** | Few, strong suggestions with reasons | 40 "recommended" badges |
| **Voice and touch are equals** | Every voice action has a tap path and vice versa | Voice-only or touch-only features |
| **Respect the body and culture** | Neutral, inclusive language and imagery | Diet-culture shaming, exoticizing food |

## 3. Visual language

- **Palette:** golden-hour kitchen — saffron, paprika, basil, cream, charcoal ink; jewel accents for occasions (plum for celebration, teal for discovery).
- **Typography:** a soft, characterful serif for display (Fraunces, variable, OFL) + a clean humanist sans for UI (Inter, OFL).
- **Shapes:** generous radii, plates and bowls as circular motifs; the Table view is literally round.
- **Illustration:** hand-drawn food "stickers" and avatar illustrations (diverse, customizable, no photos of children); photography for dishes when merchants provide it, with consistent warm grading.
- **Iconography:** rounded line icons (e.g., Phosphor) + channel glyphs (WhatsApp, SMS, phone, POS) for kitchen check status.
- **Motion:** springy and physical (plates slide onto the table, chips pop), 200–350 ms; reduced-motion alternative for everything.

## 4. The Host (companion)

An animated orb of warm steam that represents the assistant:

| State | Animation |
|---|---|
| Idle | Slow breathing glow |
| Listening | Ripples reacting to mic amplitude |
| Thinking | Gentle swirl (never > 2 s without visible progress elsewhere) |
| Speaking | Pulses with TTS amplitude |
| Celebrating (scenic only) | A tiny sparkle burst |
| Care mode | Softer color, slower breathing |

Built in Rive with a state machine driven by voice/AI events; static SVG fallback.

## 5. Lanes as layouts

| Lane | Feel | Layout |
|---|---|---|
| **Express** | Efficient, quiet | Context strip + 3 large cards + one-tap order; chat collapsed |
| **Guided** | Friendly conversation | Chat + proposal carousel + Table view toggle |
| **Scenic** | Immersive, story-led | Full-bleed imagery, course timeline, route ribbon with stops, stories inline |

Transitions between lanes animate smoothly when dials cross thresholds (with hysteresis so it never flickers).
