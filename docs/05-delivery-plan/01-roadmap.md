# Plan/01 · Roadmap

> Status: Draft v1 · Durations in weeks from kickoff (week 0) assuming a core team of ~5 engineers + 1 designer + 1 PM/ops + part-time advisors.

## 1. Phases

| Phase | Weeks | Theme | Ships |
|---|---|---|---|
| **1 · Concierge Alpha** | 0–17 | Prove the conversation and the confirm-before-pay loop | Web + desktop (Tauri); EN/ES; avatars & memory; council (F0–F12 core); curated 30–60 restaurants; WhatsApp + SMS checks; Stripe; single-restaurant orders via DoorDash Drive / Uber Direct (via Nash or direct); push-to-talk voice; closed beta in launch zone |
| **2 · Multi-Kitchen** | 17–29 | Mix kitchens, go mobile, go public in metro 1 | Synchronized legs (2–3 merchants); POS integrations (Square → Toast → Clover); iOS/Android Tauri builds; hands-free voice (LiveKit); group links; natural-medicine content GA (expert-reviewed); AI voice-call escalation; public launch |
| **3 · Concierge Runs** | 29–45 | One courier, many stops, shop extras | Partner courier pool + run cards (Stripe Issuing); shop extras with caps & substitutions; Toronto + FR-CA; calendar integrations; rituals & reminders (opt-in); membership |
| **4 · Expansion** | 45+ | More supply, more places | More NA metros; Uber Consumer Delivery API (if granted); LATAM (DR: Uber Direct, PedidosYa Courier API); B2B (offices, hotels); alcohol via licensed partners |

## 2. Gates

| Gate | Between | Criteria |
|---|---|---|
| **G1** | Phase 1 → 2 | 0 hard-constraint violations in beta; order success ≥ 95 %; availability answers p90 ≤ 5 min; proposal acceptance ≥ 40 %; DaaS production access granted; safety eval 100 % |
| **G2** | Phase 2 → 3 | Synchronized legs arrive within the 10-min window ≥ 90 %; positive contribution margin per order in the test cohort; App Store/Play approvals |
| **G3** | Phase 3 → 4 | Concierge run cost ≤ equivalent synchronized legs for ≥ 60 % of multi-stop orders; courier compliance sign-off; Canada privacy review done |

## 3. Phase 1 milestones

| Milestone | Weeks | Outcome |
|---|---|---|
| M0 Foundations | 0–2 | Monorepo, CI, auth, DB, contracts, design tokens, Tauri shell, Temporal dev |
| M1 People & Context | 2–4 | Avatars, constraints, places/GPS, Situation Brief, dial engine, ruleset loader |
| M2 Catalog & Ingestion | 4–7 | Partner console v0, menu upload → AI parse → review → publish, canonical dishes, enrichment, hybrid search |
| M3 Conversation & Council | 7–10 | Host loop + tools, faculties, synthesizer, composer, proposals, Table view, Why-this |
| M4 Liaison & Ordering | 10–13 | Basket, WhatsApp/SMS checks, payments, firm confirm, merchant payment, single dispatch, tracking, feedback, memory curator |
| M5 Voice & Polish | 13–15 | Voice gateway (push-to-talk), Host orb, express lane, accessibility pass, Spanish |
| M6 Beta Hardening | 15–17 | Full evals, load tests, security review, ops console, runbooks, closed beta (≈ 200 households) |

Parallel non-engineering track (from week 0): DoorDash Drive & Uber Direct sandbox → production review; Nash contract; Stripe (Connect, Issuing, Tax) applications; WhatsApp Business verification + template approvals; 10DLC registration; legal (terms, privacy, consumer health data policy); advisors (RD, naturopath/TCM, allergist, cultural consultants); restaurant outreach in the launch zone.
