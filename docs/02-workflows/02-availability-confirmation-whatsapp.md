# WF/02 · Merchant Availability & Confirmation (WhatsApp automation)

> Status: Draft v2 (revised 2026-10-15: WhatsApp ordering templates confirmed by founder; opt-in capture; courier-pays template) · Decision: [ADR-0007](../01-architecture/adr/ADR-0007-multichannel-merchant-liaison.md) · Policy facts verified 2026-10-15 ([research §4](../00-product/02-research-and-partner-landscape.md#4-messaging-whatsapp-sms-voice))

## 1. Goal

Within minutes, get a reliable answer from every kitchen in the basket: *can you prepare these items by this time for courier pickup, at these prices?* Then convert the soft "yes" into a firm, paid order with zero ambiguity.

## 2. Channels and order of preference

| Channel | When | Latency | Notes |
|---|---|---|---|
| **POS** (Square/Toast/Clover) | Merchant connected POS | Seconds | Live item availability; order injection for firm confirm |
| **WhatsApp** | Merchant opted in to WhatsApp | 30 s – 3 min typical | Interactive buttons; strongest with diaspora-owned restaurants |
| **SMS** | Opted in (10DLC registered; CASL in Canada) | 1–4 min | Numeric replies ("1 = yes") |
| **AI voice call** | Escalation, consented at onboarding | 1–2 min | Short scripted call with speech understanding; transcript stored |
| **Partner app/console** | Merchant uses our tablet/console | Seconds | Push + sound on new check |
| **Ops human call** | Last resort | Variable | Ops console click-to-call |

Each merchant has an ordered `merchant_channels` list with quiet hours and opt-in evidence.

## 3. WhatsApp setup

- **WhatsApp Business Platform (Cloud API)** with a verified business, display name "NothingBetter Orders", one number per metro (local area code).
- Use Meta directly or a BSP (Twilio) — choose by ops tooling; adapter interface is the same.
- **Opt-in** captured during onboarding before any template is sent (see §3a); store timestamp, method, text version. Numbers found on websites, listings or marketplaces are **not** opt-in.
- **Template category:** *utility* (order-related). Templates in EN and ES (FR in Canada).
- **24-hour window:** the first check to a merchant uses a template; once the merchant replies, follow-ups in the next 24 h are free-form/interactive (free). Utility templates sent inside an open window are free; outside, billed per delivered template.
- **Escalation path:** every automated thread offers "Call NothingBetter" / human transfer (policy requirement).
- **Scope:** business-specific order workflow only (allowed under Meta's 2026 policy that bans general-purpose AI assistants on the Business API).

## 3a. Opt-in capture (how a kitchen gets onto WhatsApp ordering)

| Method | Flow | Evidence stored |
|---|---|---|
| **Inbound-first** (preferred) | Ops calls/visits → sends or shows a `wa.me/<our-number>?text=JOIN%20NB-<code>` link or QR → kitchen sends "JOIN" | The inbound message (timestamp, number, code) — also opens a free 24-h window for onboarding questions |
| Partner console | Checkbox "Send me order requests on WhatsApp at +1…" with policy text and version | Checkbox event, IP, user, text version |
| Paper/tablet at field visit | Signed onboarding form; ops enters it in console | Scan + console record |

Cold outreach by WhatsApp template to numbers from websites, Google listings or marketplaces is not allowed (WhatsApp policy) — first contact happens by phone, visit or email.

## 3b. Ordering by template (the founder's flow)

The availability check **is** the pickup order request and the firm confirmation **is** the order; for cluster runs a third template tells the kitchen that our courier pays at the counter. Sequence: `availability_check_v1` → (customer pays) → `firm_confirmation_v1` or `courier_paid_pickup_v1` → `ready_ping_v1` → courier arrives.

## 4. Templates (drafts)

```yaml
# knowledge/templates/merchant/whatsapp/availability_check_v1.yaml
name: availability_check_v1
category: UTILITY
languages: [en_US, es_US]
body_en: >
  Hi {{1}}, this is NothingBetter. Can you prepare this pickup order by {{2}}?
  Order {{3}}: {{4}}. Total at menu prices: {{5}}. Paid by NothingBetter before pickup.
buttons:
  - { type: QUICK_REPLY, text: "✅ Yes, all good" }
  - { type: QUICK_REPLY, text: "⚠️ Changes" }
  - { type: QUICK_REPLY, text: "❌ Can't do it" }
footer: "Reply HELP for a person. STOP to opt out."
```

```yaml
name: firm_confirmation_v1
category: UTILITY
body_en: >
  Confirmed ✅ Order {{1}} — {{2}}. Please have it ready by {{3}} for courier pickup ({{4}}).
  {{5}}            # allergy line, e.g. "⚠️ SEVERE SESAME ALLERGY: Mia's kids pasta — no sesame, clean utensils."
  Payment: {{6}}   # "Paid via NothingBetter account" | "Card call from NothingBetter within 5 min"
buttons:
  - { type: QUICK_REPLY, text: "👍 Got it" }
  - { type: QUICK_REPLY, text: "Problem" }
```

```yaml
name: courier_paid_pickup_v1
category: UTILITY
body_en: >
  Confirmed ✅ Order {{1}} — {{2}}. Ready by {{3}} please.
  Our courier {{4}} will pay {{5}} at your counter by card. {{6}}
buttons:
  - { type: QUICK_REPLY, text: "👍 Got it" }
  - { type: QUICK_REPLY, text: "Problem" }
```

Others: `ready_ping_v1` ("Is order {{1}} ready? Courier ETA {{2}}"), `courier_arriving_v1`, `order_cancelled_v1`, `substitution_question_v1`.

Short order codes (`NB-4F2`) appear in every message for merchants handling several orders.

## 5. Conversation handling

```text
inbound message (webhook) ─▶ verify signature ─▶ persist raw ─▶ match to open check
   match by: context.message_id (reply-to) > order code in text > most recent open check for this merchant
   ├─ button payload → deterministic result
   ├─ text / voice note → transcribe (voice) → Liaison parser (Claude, structured output):
   │      { decision: yes|changes|no|question, items:[{name, available, substitute?}],
   │        readyAt?, priceChanges?, message_to_customer?, needs_human? }
   ├─ "changes" → interactive list message per item (in-window) → collect specifics
   └─ unclear / angry / off-topic → needs_human → ops queue + polite holding reply
signal AvailabilityCheckWorkflow(legId, result)
```

### `AvailabilityReply` (structured output)

```ts
const AvailabilityReply = z.object({
  decision: z.enum(["yes", "changes", "no", "question", "unclear"]),
  readyAtLocal: z.string().nullable(),          // "19:40"
  items: z.array(z.object({
    lineId: z.string(),                          // our basket line ids sent in the message
    available: z.boolean().nullable(),
    substitute: z.string().nullable(),
    priceMinor: z.number().nullable(),
  })),
  noteForCustomer: z.string().nullable(),
  needsHuman: z.boolean(),
  language: z.enum(["en", "es", "fr", "other"]),
  confidence: z.number(),
});
```

Low confidence (< 0.8) on a `yes` → ask a confirmation button ("Just to confirm: all items by 7:40?").

## 6. Escalation ladder (per check)

| T+ | Action |
|---|---|
| 0:00 | Primary channel (POS → WhatsApp → SMS per merchant config) |
| 2:00 | Reminder on same channel (interactive if window open) |
| 3:00 | Secondary channel (e.g., SMS if WhatsApp unread; WhatsApp if SMS) |
| 4:00 | AI voice call (consented) — "Hi, NothingBetter here with a pickup order for 7:40…" |
| 6:00 | Ops queue (human call) + user sees "Still waiting on Taquería Luna — want the backup?" |
| 8:00 | Global timeout → leg marked `NO_ANSWER` → backup offered |

Timers adapt to urgency (express → shorter) and merchant reliability history.

## 7. Firm confirmation & payment message

After the customer authorizes:
1. `firm_confirmation_v1` with allergy line and payment method.
2. If payment is by **single-use virtual card**: ops or an automated secure flow provides the card to the merchant (phone call or the merchant's own online ordering with card entry by our automation where permitted). The card is limited to the leg amount + 5 %, food merchant categories, and expires in 6 h.
3. Merchant acknowledges → leg `CONFIRMED`; ready pings at `ready_by − 5 min` if no "ready" signal.

## 8. Merchant experience principles

- **Respect their rush:** one message per check, buttons first, no chit-chat; never message during declared quiet hours.
- **Make yes easy:** prices/menu items exactly as their menu; short codes; totals; "paid before pickup".
- **Avoid wasted checks:** checks only on explicit user intent ("Check & hold"); track check-to-order conversion per merchant; cap checks/hour per merchant.
- **Language:** merchant's preferred language (EN/ES/FR).
- **Learn:** their typical answer time, peak hours, items often unavailable → better scheduling and backups.

## 9. Metrics

Answer rate within 5 min, median answer time, check-to-order conversion, parse accuracy, escalation frequency, opt-out rate, template quality rating, cost per check (templates billed + calls).
