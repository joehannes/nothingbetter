# Design/05 · Accessibility & Internationalization

> Status: Draft v1 · Target: WCAG 2.2 AA across web, desktop and mobile.

## 1. Accessibility requirements

| Area | Requirement |
|---|---|
| Keyboard | Full keyboard operation on web/desktop; visible focus (`--nb-focus`); `Space` push-to-talk only when focus isn't in a text field |
| Screen readers | Semantic structure; ARIA live regions for streaming replies, kitchen check updates, order status; proposal cards as articles with headings |
| Drag & drop (Table view) | Accessible alternative: "Move to seat…" menu (WCAG 2.2 dragging movements) |
| Color & contrast | AA contrast; no color-only status; badges with icons + text |
| Motion | Reduced-motion variants; no auto-playing video; celebrations optional |
| Text size | Up to 200 % without loss; dynamic type on iOS/Android |
| Targets | ≥ 44×44 pt; spacing between chips |
| Time limits | Kitchen hold countdown extendable ("Need more time?" → re-check) — WCAG timing adjustable |
| Voice | Captions for all TTS; adjustable speech rate; text alternative always |
| Cognitive | Plain-language mode (shorter sentences, fewer options); consistent layouts; undo for edits |
| Forms | Labels, error messages tied to fields, input purposes (autocomplete) |

Testing: axe in CI (Storybook + Playwright), manual VoiceOver/TalkBack/NVDA passes per release, users with disabilities in research rounds.

## 2. Internationalization

| Topic | Approach |
|---|---|
| Languages | EN-US, ES-US (launch); FR-CA, EN-CA (Canada phase); more by metro |
| Messages | ICU MessageFormat (FormatJS); no string concatenation; pluralization & gender-neutral phrasing |
| AI output | Host replies in user's language; knowledge entries have multilingual labels; dish names keep original language + explanation |
| Formats | `Intl` for numbers, currency (USD/CAD), dates, times (12/24 h by locale) |
| Units | Imperial (US) / metric (CA) for distance; nutrition always in g/mg with kcal |
| Addresses | Country-specific forms (US ZIP, CA postal code; unit/apartment conventions) |
| Calendars | Holidays per country/province/state; religious calendars as opt-in observances |
| Fonts | Fraunces/Inter cover Latin Extended; plan fallbacks for future scripts |
| RTL | Not in v1; layout uses logical CSS properties to keep RTL possible |

## 3. Inclusive content

- Diverse avatar illustration set (skin tones, hair, head coverings, mobility aids, ages).
- Food imagery reflecting many cultures without exoticizing captions.
- Gender-neutral defaults ("they") unless the user specifies otherwise for an avatar.
