# Design/02 · Design System

> Status: Draft v1 · Implemented in `packages/ui` · Tokens as CSS variables + Tailwind v4 theme · Light & dark themes.

## 1. Color tokens

| Token | Light | Dark | Use |
|---|---|---|---|
| `--nb-bg` | `#FFF8EE` (cream) | `#17130F` | App background |
| `--nb-surface` | `#FFFFFF` | `#221C17` | Cards, sheets |
| `--nb-surface-2` | `#F7EBDA` | `#2C241D` | Secondary surfaces, chips |
| `--nb-ink` | `#1F1A17` | `#F6EEE4` | Primary text |
| `--nb-ink-muted` | `#6B5E54` | `#BBAE9F` | Secondary text |
| `--nb-line` | `#E8D9C4` | `#3A3029` | Borders, dividers |
| `--nb-saffron` | `#E89A2C` | `#F2B154` | Primary brand / Host |
| `--nb-paprika` | `#C8472A` | `#E26A4C` | Primary action (Order, Pay) |
| `--nb-basil` | `#3E7A4D` | `#6FB07F` | Success, "safe for" badges |
| `--nb-plum` | `#6A3A8C` | `#A57BC6` | Celebration accent |
| `--nb-teal` | `#1F7A7A` | `#4DB6B6` | Discovery/inspiration accent |
| `--nb-warning` | `#B7791F` | `#E3A84A` | Cautions |
| `--nb-critical` | `#B42318` | `#F97066` | Allergy blocks, errors |
| `--nb-focus` | `#2563EB` | `#93B4FF` | Focus ring (always visible) |

Contrast: text tokens on backgrounds ≥ 4.5:1 (AA); large display text ≥ 3:1. Status is never color-only — always icon + label.

## 2. Typography

| Token | Font | Size / line | Use |
|---|---|---|---|
| `display-xl` | Fraunces 600 (opsz auto, soft) | 40/44 | Scenic hero titles |
| `display-l` | Fraunces 600 | 32/38 | Section heroes |
| `title` | Inter 650 | 20/28 | Card titles |
| `body` | Inter 400 | 16/24 | Default text |
| `body-s` | Inter 400 | 14/20 | Meta (ETA, price) |
| `caption` | Inter 500 | 12/16 | Badges, captions |
| `mono-num` | Inter tabular numbers | — | Prices, times |

Dynamic type: scales with OS/browser settings up to 200 % without loss of function.

## 3. Spacing, radius, elevation

- Spacing scale: 4, 8, 12, 16, 24, 32, 48, 64.
- Radius: `sm 8`, `md 14`, `lg 20`, `xl 28`, `round 999` (chips, plates).
- Elevation: 3 levels with warm-tinted shadows (light) / lighter surfaces (dark).
- Touch targets ≥ 44×44 pt.

## 4. Motion tokens

| Token | Value | Use |
|---|---|---|
| `motion-fast` | 150 ms, ease-out | Chips, toggles |
| `motion-base` | 250 ms, spring (stiffness 300, damping 26) | Cards, sheets |
| `motion-slow` | 400 ms | Lane transitions |
| `motion-celebrate` | 600 ms | Scenic-only flourishes |

All motion disabled or replaced by fades under `prefers-reduced-motion`.

## 5. Core components

| Component | Description | Key states |
|---|---|---|
| `ContextStrip` | Chips for time, place, table, occasion, budget; tap to edit | Detected / edited / unknown |
| `HostOrb` | Rive companion + mic control | idle, listening, thinking, speaking, care, celebrate, muted, permission-denied |
| `Composer` | Text input + mic + quick chips | typing, recording (waveform), sending |
| `QuickChips` | Answer chips; dial chips (*Just get me food*, *Inspire me*, *Keep it healthy*, *Take me on a journey*) | single/multi select |
| `ProposalCard` | Dish or meal-set card: image, name, merchant, price, ETA, badges, one-line why | default, selected, unavailable, swapped |
| `MealSetCard` | Seats row with dish thumbnails, total, couriers count, ETA window | — |
| `TableView` | Round table with avatar seats; drag dishes to seats; fit badges per seat | editing, read-only, group link |
| `SeatBadge` | ✓ sesame-safe (verified) · ✓ halal · ♥ loved before · ✦ new for you | verified vs estimated styles |
| `KitchenCheckChips` | Per-merchant live status with channel glyph | sending, delivered, read, yes, changes, no, escalated, backup |
| `RouteRibbon` | Ordered stops with icons and ETAs | planned, live |
| `TrackingMap` | MapLibre map with courier(s) and stops | live, stale, delivered |
| `WhyThisSheet` | Reasons, citations, evidence levels, memory used | — |
| `MemoryItem` | Fact with provenance, confidence, edit/delete | hard fact, preference, episode |
| `PriceBreakdown` | Per-leg subtotal, fees, tax, tip, extras cap | estimate, final |
| `SafetyNotice` | Calm, high-contrast notice with resources | info, warn, critical |
| `AvatarBuilder` | Illustrated avatar creator | — |

## 6. Badges & evidence styling

- **Verified** (merchant-declared): solid badge with check icon.
- **Estimated** (AI-inferred): outlined badge with "≈" and "estimated" in accessible label.
- **Evidence level** on health/natural-medicine notes: small A/B/C/D pill with tooltip explaining the grade.

## 7. Implementation notes

- Components are headless-first (Radix) with our styling; each has Storybook stories with light/dark/RTL-safe (future) and reduced-motion variants.
- Tokens exported as CSS variables; Tailwind v4 `@theme` maps to them; Tauri and web share them.
- Visual regression via Playwright screenshots on Storybook.
