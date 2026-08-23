---
name: jobox-ai-collect-payment-and-receipt
description: Record a payment against a completed Jobox job, issue the customer receipt, and read the payment log. Covers what the Jobox Kili contract does and does not let you take back.
api: Jobox Kili API
base_url: https://api.jobox.ai/Kili
operations:
  - readPayment
  - create_13
  - update_4
  - update_5
  - create_11
  - create_12
  - delete_1
  - readPaymentLog
  - create_14
  - read_13
  - latestReceipt
  - getFeedback
  - read_4
  - list
generated: '2026-08-23'
method: generated
source: openapi/jobox-ai-kili-openapi.json + conventions/jobox-ai-conventions.yml
---

# Take a payment and issue a receipt on Jobox

## Steps

1. **Check what is owed.** `GET /jobs/{job_id}/payment` (`readPayment`) returns the payment
   state for the job.
2. **Record the payment.** `POST /payments` (`create_13`, body `Payment`). Amend with
   `PUT /payments` (`update_4`) or `PUT /payments/{payment_id}` (`update_5`).
3. **Write the payment log entry.** `POST /paymentlog` (`create_11`, body `PaymentLogDTO`).
   Read it with `GET /jobs/{job_id}/payment/log` (`readPaymentLog`) or query with
   `POST /paymentlog/list` (`create_12`).
4. **Issue the receipt.** `POST /receipts/new` (`create_14`, body `ReceiptRequestDTO`). Fetch
   it with `GET /receipts/v2/{receipt_key}` (`read_13`) or get the job's latest with
   `GET /jobs/{job_id}/receipts` (`latestReceipt`).
5. **Capture customer feedback on the receipt.**
   `POST /receipts/{receipt_key}/feedback` (`getFeedback`, body `JobFeedback`).
6. **Watch for disputes.** `GET /disputes/users/{user_id}` (`list`) and
   `GET /disputes/{dispute_id}` (`read_4`).

## Reversibility — read this before you charge anything

The contract models `Payment`, `PaymentLog`, `CardPaymentDetails` and a Stripe linkage, and
**exposes no refund, void, capture-reversal or chargeback operation at all.** The only
payment writes are `POST /payments`, `PUT /payments` and `PUT /payments/{payment_id}`.

- `DELETE /paymentlog` (`delete_1`) removes a **log entry**. That is bookkeeping. It is not a
  card refund and there is no published statement that it moves money.
- Jobox publishes **no refund window, no cancellation deadline and no capture rule.** None is
  asserted here because none exists to cite.
- **Consequence for an agent: treat every payment call as irreversible through this API.** If
  money needs to come back, that is an out-of-band process with Jobox/Talus Pay support, not
  an API call.

## Other rules

- No idempotency key exists. A retried `POST /payments` risks a duplicate charge record —
  this is the highest-consequence retry in the whole API. Always read back with
  `GET /jobs/{job_id}/payment` before retrying.
- No dry-run mode.
- Errors return `{"title","message","debug_id?"}` — not RFC 9457 `application/problem+json`.
- Card processing moved to the Talus Pay parent after the January 2024 acquisition; the fee
  schedule in the help center describes Talus Pay rates, not a published Jobox plan.
