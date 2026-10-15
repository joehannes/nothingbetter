# ADR-0008 · Check availability before payment, with soft holds

- Status: Accepted (2026-10-15)

## Context
The founder wants every item confirmed before the customer pays. Merchants dislike checks that don't turn into orders.

## Decision
- Default flow: basket accepted → availability checks (soft hold, e.g., 10 min) → customer pays (authorization) → firm confirmation → merchant payment → dispatch.
- Only send checks when purchase intent is explicit ("Check & hold" tap).
- POS-integrated merchants skip the human check (live availability).
- Variant (feature flag `auth_then_check`): authorize first, then check, for returning users with saved cards — voids the authorization if anything fails.
- Track merchant "check-to-order" conversion and throttle checks per merchant.

## Consequences
- + No charges for unavailable food; fewer refunds.
- − Adds 1–5 minutes before payment for non-POS merchants; mitigated by parallel fan-out, live progress UI, and pre-computed substitutes.
