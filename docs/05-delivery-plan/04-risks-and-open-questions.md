# Plan/04 · Risks & Open Questions

> Status: Draft v1

## 1. Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | DaaS production access delayed/denied (DoorDash Drive review; Uber Direct written approval) | Med | High | Apply week 0; Nash as aggregator; keep both adapters; local courier partner as fallback |
| R2 | No native multi-pickup single-drop at DaaS | High | Med | Synchronized legs (Phase 2); concierge runs (Phase 3); verify Nash batched jobs |
| R3 | Restaurants slow to answer checks at rush hour | High | High | POS integrations; partner console alerts; escalation ladder; backups; reliability scoring; prefer partners |
| R4 | Multi-leg fees make orders expensive | Med | High | Fee-aware composer; prefer single-kitchen sets when equal; concierge runs in dense zones; transparent trade-offs |
| R5 | Allergen error reaching a diner | Low | Critical | Layered guardrails; verified-only for severe allergies; incident process; insurance |
| R6 | Health-data law exposure (e.g., WA MHMDA private right of action) | Med | High | Explicit consent, minimization, geofencing review, counsel sign-off |
| R7 | Natural-medicine claims challenged (FTC) | Med | Med | Evidence grades, banned phrases, expert review |
| R8 | WhatsApp pricing/policy changes (rate updates Oct 2026; reported business-agent token pricing) | Med | Med | Channel abstraction; SMS/voice fallback; monitor Meta notices |
| R9 | AI cost per order too high | Med | Med | Effort tuning, caching, Batch for background, cost dashboards, route-level budgets |
| R10 | AI latency hurts express lane | Med | Med | Low effort, cached prompts, precomputed candidates, streaming UI |
| R11 | Cultural stereotyping / offense | Low | High | Self-declared only; community consultants; audits; feedback channel |
| R12 | Tauri platform quirks (Linux mic, iOS audio, background location) | Med | Med | Native audio plugin; vendor driver app for couriers; early device testing |
| R13 | Gig-worker law compliance for own couriers | Med | High | Partner courier company first; counsel before own pool |
| R14 | Sales-tax complexity per jurisdiction | Med | Med | Stripe Tax/Avalara; start in one state |
| R15 | Merchant fatigue from checks that don't convert | Med | Med | Checks only on explicit intent; per-merchant throttles; conversion tracking |

## 2. Open questions (owner decisions)

| # | Question | Recommendation |
|---|---|---|
| Q1 | **Launch metro** | Miami-Dade (bilingual, WhatsApp-heavy, Latin/Caribbean fit); Toronto second |
| Q2 | Brand name: keep "NothingBetter"? Domain? | Keep as working name; check trademark and domains |
| Q3 | Natural-medicine lens default on or off at onboarding? | Suggest on with one-line explanation; user chooses |
| Q4 | Delivery: Nash orchestrator vs. direct DoorDash Drive + Uber Direct only | Start with Nash if commercial terms work; keep direct adapters |
| Q5 | Courier model for concierge runs | Partner courier company first |
| Q6 | Service-fee model and membership | Model in Phase 1 beta; decide before public launch |
| Q7 | Cheaper model for bulk menu extraction | Decide after M2 evals (Opus 5.5 at lower effort vs Haiku 5.5) |
| Q8 | STT/TTS vendors | Decide after voice eval in M5 |
| Q9 | Meaning of "bid from various restaurants" | Assumed: order from several restaurants in one basket. Optional later feature: "Ask the kitchens" requests where restaurants send offers (reverse bidding) |
| Q10 | Calendar integration scope (Google/Microsoft/Apple) | Phase 3, read-only, opt-in |
