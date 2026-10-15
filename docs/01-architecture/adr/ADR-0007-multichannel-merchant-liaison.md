# ADR-0007 · Multi-channel merchant liaison, WhatsApp-first where opted in

- Status: Accepted (2026-10-15)

## Context
We must confirm availability with merchants quickly. WhatsApp is preferred by the founder and popular with many diaspora-owned restaurants; many US/CA restaurants respond faster to SMS or phone; some have POS integrations.

## Decision
`MerchantChannel` interface with adapters: `pos` (instant, preferred when available), `whatsapp` (Cloud API, utility templates + interactive buttons), `sms` (Twilio, 10DLC), `voice` (AI call via Twilio + STT/TTS with consent), `partner_app` (console/tablet), `email` (fallback). Each merchant has an ordered channel list and opt-in evidence. Escalation ladder with timers per check.

## Consequences
- + Higher answer rates; no single-channel outage stops ordering.
- − More templates and compliance surfaces (WhatsApp, 10DLC, TCPA/CASL).
