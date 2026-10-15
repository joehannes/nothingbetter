# WF/04 · Payments & Money Flow

> Status: Draft v1 · Decision: [ADR-0006](../01-architecture/adr/ADR-0006-merchant-of-record-direct-merchant-payment.md) · ⚖️ Validate MoR, tax and money-transmission questions with counsel and Stripe.

## 1. Model

**NothingBetter is the merchant of record** for the customer's order. We buy food from merchants and delivery from providers, and we resell a complete, delivered meal.

```text
Customer ──(card/Apple Pay/Google Pay)──▶ NothingBetter (Stripe PaymentIntent, manual capture)
NothingBetter ──▶ Restaurant A (Connect settlement)        ──▶ food
NothingBetter ──▶ Restaurant B (single-use virtual card)    ──▶ food
NothingBetter ──▶ DaaS provider (per-delivery fee)          ──▶ delivery
NothingBetter ──▶ Courier run card (Issuing, capped)        ──▶ shop extras / courier-paid pickups (concierge runs)
NothingBetter ──▶ Own couriers (Connect payouts: run pay + 100 % tips)
NothingBetter ──▶ Tax authorities (sales tax remitted)
```

The founder's original idea ("pay the delivery service all the money; the courier pays each restaurant") is implemented **only** where couriers are ours or a partner's (concierge runs), via a capped card — never cash, never with DaaS couriers (they don't support it).

## 2. Customer charge

| Component | Notes |
|---|---|
| Food subtotal (per leg) | Menu prices as confirmed by merchants |
| Shop extras **cap** | Authorized at cap; captured at actual |
| Delivery fee | Pass-through of quoted provider fees (per leg) or concierge-run fee |
| Service fee | % of subtotal with min/max (configurable per metro) |
| Small-order fee | Below threshold (configurable) |
| Sales tax | Per jurisdiction, computed per leg (Stripe Tax/Avalara) |
| Courier tip | 100 % to couriers |
| Credits/promos | Applied before tax where required |

Total shown **before** checks (estimate) and **after** checks (final); price changes within tolerance (≤ 5 % or ≤ $2 per leg) absorbed or flagged per policy.

## 3. Authorization & capture

1. `PaymentIntent` with `capture_method=manual`, amount = final total + extras cap.
2. Authorization happens **after** all legs are available (check-then-pay, [ADR-0008](../01-architecture/adr/ADR-0008-check-then-pay.md)).
3. Capture at delivery with actual amounts (≤ authorized). Partial capture releases the remainder.
4. If an increase is needed (user adds items after authorization) → new authorization for the delta (or incremental authorization where the card network supports it).
5. Authorizations expire (card-network dependent, typically days) — orders are same-day, so not an issue; scheduled orders > 24 h ahead authorize on the day (card saved via SetupIntent).

## 4. Paying merchants

| Method | Who | Flow | Timing |
|---|---|---|---|
| **Connect settlement** | Partner restaurants/shops with Stripe Connect (Express) accounts | Ledger payable per leg → transfer on payout schedule (e.g., daily) | Merchant accepts our order as prepaid |
| **Single-use virtual card** | Listed (non-partner) restaurants | Create Issuing card (amount × 1.05, food MCCs, 6 h expiry) → give card to merchant by phone/secure link or use the merchant's own online ordering checkout | At firm confirmation |
| **Courier run card** | Concierge runs (and shop extras) | Card limit = Σ planned courier-paid amounts × 1.10; real-time auth checks | At pickup/purchase |
| **POS-integrated prepaid** | Merchants whose POS supports third-party prepaid orders | Order injected as paid by NothingBetter; settlement via Connect | At firm confirmation |

## 5. Ledger (double-entry)

Every money movement writes balanced entries in `payments.ledger_entries`:

| Event | Debit | Credit |
|---|---|---|
| Capture $62.40 | customer_receivable | revenue_food $48.00, revenue_delivery $7.99, revenue_service_fee $2.90, tax_payable $3.51 |
| Merchant A payable | cost_of_food | merchant_payable:rst_A |
| Virtual card spend at B | cost_of_food | issuing_balance |
| DaaS fee | cost_of_delivery | provider_payable:doordash |
| Tip | customer_receivable | courier_payable |
| Refund $9 | refunds | customer_receivable |

Daily reconciliation job: Stripe balance transactions ↔ ledger ↔ provider invoices; mismatches → ops queue.

## 6. Refunds & credits

| Case | Default |
|---|---|
| Missing item (customer report, ≤ $25) | Instant partial refund or credit (customer's choice) |
| Wrong item | Refund item + optional redelivery if feasible |
| Very late (> 20 min past window) | Delivery-fee credit |
| Allergy-related error | Full refund + sev-1 incident + follow-up |
| Merchant cancellation after payment | Full refund of that leg + delivery fee; backup offered |

Recovery from merchants (Connect reversal / dispute) handled by ops with evidence (photos, messages).

## 7. Fraud & risk

- Stripe Radar rules; 3DS when risk is elevated.
- Velocity limits for new accounts (order value, number of extras).
- Issuing: per-authorization limits, allowed categories, merchant country = US/CA, expiry; real-time approval checks order context.
- Courier receipt verification; anomaly detection on run card spend.
- Promo abuse limits per device/payment fingerprint.

## 8. Pricing configuration (per metro)

```yaml
metro: us-fl-miami
currency: USD
service_fee: { percent: 8, min_minor: 199, max_minor: 799 }
small_order: { threshold_minor: 1500, fee_minor: 299 }
price_change_tolerance: { percent: 5, absolute_minor: 200 }
extras: { max_item_minor: 5000, max_total_minor: 8000, buffer_percent: 10 }
virtual_card_buffer_percent: 5
tax_provider: stripe_tax
```

(Values are placeholders for modeling, not decisions.)
