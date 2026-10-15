# 06 · Security, Privacy & Compliance

> Status: Draft v1 · Not legal advice — have counsel review before launch. Items marked ⚖️ need a legal sign-off.

## 1. Data classification

| Class | Examples | Controls |
|---|---|---|
| **S3 – Sensitive** | Health conditions, medications, pregnancy, religious observance, ethnicity/heritage, children's data, precise location history | Opt-in consent per category; app-layer encryption (🔒 columns); access only by `people` module; excluded from analytics and model-training exports; purge on consent withdrawal |
| **S2 – Personal** | Name, phone, email, addresses, order history, taste preferences, transcripts | Encrypted at rest (RDS/KMS), row-level access by owner, retention limits |
| **S1 – Business** | Merchant data, menus, prices, provider references | Standard controls |
| **S0 – Public** | Marketing content, public knowledge entries | — |

## 2. Privacy by design

1. **Self-declared sensitive attributes only.** Culture, religion, health are entered by the user (or caregiver for a minor). Never inferred from name, voice, face, location, or orders ([ADR-0010](adr/ADR-0010-no-sensitive-inference.md)).
2. **Purpose limitation.** Health and culture data used only to tailor food suggestions and safety checks; never for ads, pricing, or sold/shared.
3. **Consent ledger.** `identity.consents` records category, version of notice, timestamp, channel, and withdrawal.
4. **Transparency UI.** "What I know about you" shows every memory and preference with provenance and delete/edit.
5. **Avatars of other people.** A user may create profiles for family/friends (shadow profiles). The owner attests they have permission; adults can be invited to claim and control their own avatar; sensitive data on others' avatars limited to what's needed for safety (allergies, diet rules).
6. **Minors.** The app is for adults (18+ account holders). Child avatars are caregiver-managed; no direct collection from children; no photos of minors (illustrated avatars only); no marketing to minors. ⚖️ COPPA review.
7. **Location minimization.** Precise GPS used to suggest an address, then discarded unless saved as a place. Courier tracking shown only during active orders.
8. **Voice.** Audio streamed for transcription, not stored by default; no speaker identification or voiceprints (avoids biometric-privacy laws such as Illinois BIPA).
9. **AI vendors.** Use API terms that exclude training on our data; prefer zero/limited data retention options where available; minimize PII in prompts (avatar ids + needed attributes, not full identity).

## 3. Regulatory map (US & Canada) ⚖️

| Area | Rules to address | Our approach |
|---|---|---|
| Consumer health data | Washington My Health My Data Act (private right of action), Nevada SB 370, Connecticut consumer health data provisions; CCPA/CPRA "sensitive personal information"; state privacy laws requiring opt-in for sensitive data | Separate, explicit health consent; consumer-health-data privacy policy; geofencing rules; no sale/sharing; deletion workflows |
| General privacy | CCPA/CPRA and other state comprehensive laws; PIPEDA (Canada); Quebec Law 25 (PIA for cross-border transfer, privacy officer) | DSAR tooling (access, delete, correct, portability); Canada data in Canada region |
| Health claims | FTC Act §5 (substantiation for health claims), FTC health products guidance; FDA (no disease claims) | No disease treatment/cure claims; evidence-graded statements; natural-medicine framed as tradition + evidence level; expert review |
| Health apps | FTC Health Breach Notification Rule (applies to personal health records drawing from multiple sources) | v1 does not import from fitness/health platforms; revisit before adding Apple Health / Google Health Connect |
| Messaging | TCPA, FCC AI-voice ruling (2024), A2P 10DLC (US SMS), CASL (Canada), WhatsApp Business policy (opt-in, templates, escalation) | Opt-in capture with evidence; STOP/opt-out handling; quiet hours; human escalation |
| Payments | PCI DSS 4.0 | Stripe Elements (SAQ A); Issuing card details only via Stripe Issuing Elements; no PAN storage |
| Sales tax | Marketplace facilitator / seller rules for prepared food (state & provincial) | Stripe Tax or Avalara; tax per leg & jurisdiction |
| Couriers | Gig-worker minimum pay laws (e.g., NYC app-based delivery workers, Seattle), classification rules (e.g., California) | Cluster-run pilot uses a contracted courier company or staffing-platform shifts; own contractors only after local counsel review |
| Alcohol / restricted goods | State/provincial licensing, ID verification | Excluded from v1 |
| Allergens | Restaurants generally aren't required to label allergens like packaged food | Only restaurant-declared allergens shown as facts; estimates labeled "not verified"; disclaimer + "ask the kitchen" flow |
| Accessibility | ADA (web), AODA (Ontario), WCAG 2.2 AA | See [design/05](../04-design/05-accessibility-and-i18n.md) |
| AI transparency | Bot-disclosure laws (e.g., California), emerging state AI laws | Clear "AI assistant" labeling; human escalation; merchants told they're talking to an automated assistant |

## 4. Security architecture

- **AuthN:** email/phone OTP, Sign in with Apple, Google; passkeys in Phase 2. Refresh tokens bound to device; revocation list.
- **AuthZ:** policy checks in module services (owner/household roles: owner, adult member, viewer); partner roles (owner, manager, staff); ops roles with just-in-time elevation and audit.
- **Secrets:** AWS Secrets Manager; short-lived credentials; per-provider webhook secrets rotated.
- **Network:** private subnets for DB/Redis; WAF + rate limits on public endpoints; webhook endpoints allowlisted where providers publish IP ranges.
- **Encryption:** TLS 1.2+; RDS encryption; app-layer envelope encryption for 🔒 columns (KMS data keys).
- **Tauri hardening:** strict CSP; capabilities restricted per window; no remote URLs loaded in the main webview; IPC commands allowlisted; updater signatures verified.
- **AI-specific threats:** prompt injection via merchant replies or menu text → treat all external text as data, structured outputs only, tool allowlists per agent, no tool can move money without the deterministic validator; jailbreak attempts → stay on-domain; PII leakage → redaction in logs.
- **Money controls:** spending limits on Issuing cards; velocity checks; refund limits per agent/role; four-eyes for manual payouts above threshold.
- **Audit:** append-only `ops.audit_log` for memory writes, constraint changes, money movements, ops overrides.
- **Testing:** SAST/dependency scanning in CI; quarterly pen test before launch; red-team suite for AI ([AI/11](../03-ai/11-evaluation-and-quality.md)).

## 5. Incident response

Sev-1: any hard-constraint violation reaching a diner (allergen/diet), payment over-charge at scale, data exposure. Runbooks in [07 Infrastructure & ops](07-infrastructure-and-ops.md#6-runbooks).
