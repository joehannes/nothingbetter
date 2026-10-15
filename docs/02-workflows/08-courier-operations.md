# WF/08 · Courier Operations (cluster runs)

> Status: Draft v1 · 2026-10-15 · Supports [ADR-0005](../01-architecture/adr/ADR-0005-delivery-orchestration.md) rev 2 · ⚖️ = legal review

## 1. Purpose

Run the founder's model safely: **one courier collects from several nearby kitchens, pays at each counter with a capped card, buys extras on the way, and delivers everything together.**

## 2. Crew sourcing (pilot)

| Option | Setup | Pros | Cons |
|---|---|---|---|
| **Local courier company** (recommended for pilot) | Contract with per-run or hourly rate; their drivers use our dispatch app (Onfleet) | Insured, trained, fast to start, no employment overhead | Less control; margin |
| **Shift staffing platform** (e.g., Instawork W-2 shifts via an affiliate employer of record; GigSmart) | Book drivers for peak shifts | Classification handled by platform (W-2 option) | Verify vehicle, delivery experience, insurance |
| **Own contractors** | Stripe Connect payouts per run + 100 % tips | Brand experience, control | ⚖️ City minimum-pay rules, classification, insurance; later phase |

## 3. Shifts & capacity

- Pilot: dinner 17:00–21:00 daily + weekend lunch 11:30–14:30; 2–4 couriers per shift in one zone.
- Capacity signal to the dispatcher: couriers on shift, current run ETA, location.
- Target utilization ≥ 2 runs/courier-hour at peak (drives cost per run; WF/03 §3).

## 4. Equipment & standards

- Insulated bags (hot/cold separation), drink carrier, phone mount, power bank.
- Food-safety basics: separate hot and cold, sealed bags with tamper stickers, no stacking of soups.
- Allergy handling: bags labeled by seat when a severe allergy is present; never open sealed bags.
- Professional conduct guide; bilingual phrases (EN/ES) for counters.

## 5. Run card (Stripe Issuing)

| Control | Setting |
|---|---|
| Card type | Virtual card in mobile wallet (tap-to-pay) or physical card |
| Lifetime | Issued/activated at run start, frozen at run end |
| Per-authorization limit | Planned leg amount × 1.10 |
| Daily limit | Σ planned run amounts × 1.10 |
| Allowed categories | Restaurants, fast food, grocery/convenience, bakeries, florists |
| Card presence | Card-present only |
| Real-time authorization | Approve only if merchant matches an open stop of the run (location/MCC/name heuristics) and amount within limit |
| Receipts | Photo in driver app per purchase; OCR total vs authorization; mismatch > 10 % → review |

## 6. Run lifecycle

```text
assigned → card activated → stop 1 (arrive → pay → photo receipt → seal bag → picked_up)
        → stop 2 … → shop stop (buy within caps, substitutions per rule) → en route → delivered (photo/PIN)
        → card frozen → receipts reconciled → courier rated (optional) → payout/invoice
```

Driver app events map to `DeliveryEvent` (WF/03 §5), including `purchase_made(stop, amount, receiptUrl)`.

## 7. Safety & compliance ⚖️

- Insurance: commercial auto/hired & non-owned auto coverage via the courier company or our policy; cargo coverage.
- Labor: local minimum-pay rules for app-based delivery workers (some cities have them), classification rules; tip pass-through 100 %.
- Background checks per partner policy.
- No alcohol/tobacco/pharmacy purchases in v1.
- Incident reporting: accidents, harassment, food-safety issues → ops queue, sev triage.

## 8. Metrics

Runs per courier-hour, cost per run vs DaaS-equivalent, on-time rate, wait time at counters, card declines, receipt mismatches, food-quality ratings per run, courier satisfaction.
