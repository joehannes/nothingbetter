# Plan/01 · Roadmap

> Status: Draft v2 · Revised 2026-10-15 (cluster runs moved into Phase 1 as a pilot) · Durations in weeks from kickoff (week 0), assuming a core team of ~5 engineers + 1 designer + 1 PM/ops lead + part-time advisors + a contracted courier partner for the pilot.

## 1. Phases

| Phase | Weeks | Theme | Ships |
|---|---|---|---|
| **1 · Concierge Alpha** | 0–17 | Prove the conversation, confirm-before-pay, and one-courier multi-kitchen runs | Web + desktop (Tauri); EN/ES; avatars & memory; council; curated 30–60 kitchens; WhatsApp ordering templates + SMS; Stripe; single-kitchen orders via DoorDash Drive / Uber Direct; **cluster-run pilot**: 2–3 kitchens within ~1 km, one contracted courier paying at each counter with a Stripe Issuing run card, one zone at dinner peak; push-to-talk voice; closed beta |
| **2 · Multi-Kitchen at Scale** | 17–29 | Cluster runs across the metro, extras on the way, mobile, public launch | More zones and shifts; shop extras with caps & substitutions; synchronized DaaS legs as fallback; co-located (food hall) pickups; POS integrations (Square → Toast → Clover); iOS/Android; hands-free voice (LiveKit); group links; AI voice-call escalation; natural-medicine content GA; public launch |
| **3 · Second Metro** | 29–45 | Repeat the playbook | Toronto + FR-CA (data in Canada); crew model per city; calendar integrations; rituals & reminders (opt-in); membership |
| **4 · Expansion** | 45+ | More supply, more places | More NA metros; Uber Consumer Delivery API (if granted); DoorDash/Uber multi-pickup if offered; LATAM (DR: PedidosYa courier/Uber Direct + local "mandados" couriers); B2B (offices, hotels); alcohol via licensed partners |

## 2. Gates

| Gate | Between | Criteria |
|---|---|---|
| **G1** | Phase 1 → 2 | 0 hard-constraint violations; order success ≥ 95 %; availability answers p90 ≤ 5 min; proposal acceptance ≥ 40 %; DaaS production access granted; **cluster runs: on-time ≥ 90 %, cost per run ≤ DaaS-equivalent legs for ≥ 60 % of multi-kitchen orders, receipt mismatches ≤ 2 %** |
| **G2** | Phase 2 → 3 | Positive contribution margin per order; ≥ 2 runs/courier-hour at peak; App Store/Play approvals |
| **G3** | Phase 3 → 4 | Second metro hits G1 metrics; courier compliance sign-off per city; Canada privacy review done |

## 3. Phase 1 milestones

| Milestone | Weeks | Outcome |
|---|---|---|
| M0 Foundations | 0–2 | Monorepo, CI, auth, DB, contracts, design tokens, Tauri shell, Temporal dev |
| M1 People & Context | 2–4 | Avatars, constraints, places/GPS, Situation Brief, dial engine, ruleset loader |
| M2 Catalog & Ingestion | 4–7 | Partner console v0 with WhatsApp opt-in capture, menu upload/WhatsApp menu → AI parse → review → publish, canonical dishes, enrichment, hybrid search, Places discovery |
| M3 Conversation & Council | 7–10 | Host loop + tools, faculties, synthesizer, composer with kitchen-cluster rule, proposals, Table view, Why-this |
| M4 Liaison & Ordering | 10–13 | Basket, WhatsApp ordering templates + SMS, payments, firm confirm, merchant payment, single DaaS dispatch, tracking, feedback, memory curator |
| **M4b Cluster Runs** | 12–15 | `own_fleet` adapter (Onfleet linked tasks), run cards (Stripe Issuing + real-time auth), courier receipt flow, mode selection & cost model, ops board for runs |
| M5 Voice & Polish | 13–15 | Voice gateway (push-to-talk), Host orb, express lane, accessibility pass, Spanish |
| M6 Beta Hardening | 15–17 | Full evals, load tests, security review, ops console, runbooks, closed beta (≈ 200 households) incl. cluster-run pilot zone |

**Business & operations track (from week 0):** DoorDash Drive & Uber Direct sandbox → production review (and BD asks: multi-pickup, `red_card`); Nash contract (optional); **courier partner contract + insurance**; **Stripe Issuing program approval** (critical path for run cards), Connect, Tax; Onfleet account; WhatsApp Business verification + utility templates (EN/ES); 10DLC registration; legal (terms, privacy, consumer health data policy, ADR-0011 decision); advisors (RD, naturopath/TCM, allergist, cultural consultants); restaurant outreach and onboarding visits in the pilot zone (kitchen clusters, food halls).
