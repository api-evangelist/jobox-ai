---
name: jobox-ai-settle-and-report
description: Read the Jobox Wallet, move funds, and pull the weekly settlement and reconciliation reports a demand partner needs to close its books.
api: Jobox Kili API
base_url: https://api.jobox.ai/Kili
operations:
  - read_19
  - readLog_1
  - create_21
  - create_19
  - archive_4
  - read_14
  - read_9
  - readJobsByPage
  - read_6
generated: '2026-08-23'
method: generated
source: openapi/jobox-ai-kili-openapi.json + https://www.jobox.ai/marketplace
---

# Settle and reconcile on Jobox

Jobox's demand-partner product promises weekly settlement and closing reports. These are the
operations behind that.

## Steps

1. **Read the wallet.** `GET /v3/wallet/{user_id}` (`read_19`). Use the **v3** wallet — v2
   (`/v2/wallet/...`) and the unversioned wallet paths are earlier generations of the same
   surface and are still published alongside it.
2. **Read the wallet log.** `GET /v3/wallet/{user_id}/log` (`readLog_1`).
3. **Move funds.** `POST /v3/wallet` (`create_21`).
4. **Manage payout aliases.** `POST /wallet/alias` (`create_19`) and
   `DELETE /wallet/alias` (`archive_4`).
5. **Pull the reconciliation report.**
   `GET /reports/users/{user_id}/{start_time}/{end_time}` (`read_14`) for a user, or the
   contact-scoped `v3` variants
   `GET /v3/reports/users/{user_id}/contacts/all/{start_time}/{end_time}` and
   `GET /v3/reports/users/{user_id}/contacts/{contact_ids}/{start_time}/{end_time}`.
   Time ranges are **path segments, not query parameters**.
6. **Page through jobs for the period.** `GET /users/{user_id}/jobs` (`readJobsByPage`) or
   `GET /jobs/list/{page_num}` (`read_6`) — the only paginated path in the entire API.
7. **Jobox virtual numbers.** `GET /joboxnumbers` (`read_9`) returns the masked numbers used
   for pro↔customer contact.

## Rules an agent must follow

- **A wallet transfer cannot be taken back through this API.** `POST /v3/wallet` has no
  reversal operation, no cancel and no stated window. Confirm the amount and destination
  before calling, and reconcile with `GET /v3/wallet/{user_id}/log` afterwards.
- **No idempotency key.** A retried `POST /v3/wallet` risks a duplicate transfer. On a timeout,
  read the wallet log before retrying — never retry blind.
- **Pagination is effectively absent.** Only `GET /jobs/list/{page_num}` pages, and it exposes
  no page size, cursor or total. Long reporting windows may truncate silently; prefer narrow
  date ranges and stitch them.
- **No rate-limit signal.** Pace bulk report pulls yourself.
- Timezone handling: a `Timezone` / `timezone` header appears on ten operations, including the
  report reads, and is undocumented. Send it explicitly and verify boundaries against a known
  period rather than trusting the default.
