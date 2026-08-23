---
name: jobox-ai-dispatch-a-job
description: Submit a home-services job to the Jobox Kili platform, let Jobox auto-dispatch it to a matched pro, and follow it to completion. For demand partners embedding home services into their own product.
api: Jobox Kili API
base_url: https://api.jobox.ai/Kili
operations:
  - create_9
  - create_8
  - readJobUnit
  - read_7
  - readJob
  - readAll
  - readAll_3
  - readDescription
  - readNotes
  - sendStatus
  - archive_3
generated: '2026-08-23'
method: generated
source: openapi/jobox-ai-kili-openapi.json + conventions/jobox-ai-conventions.yml + errors/jobox-ai-problem-types.yml
---

# Dispatch a job on Jobox

Jobox's marketplace product takes a job from your product, parses it, and dispatches it
automatically to a matched pro. You do not choose the pro.

## Before you start

- Base URL is `https://api.jobox.ai/Kili`.
- **You need a credential Jobox has not documented.** Anonymous calls return
  `401 {"title":"Unauthorized","message":"You cannot access this resource","debug_id":"dbg_..."}`.
  Onboarding is sales-gated — there is no self-service signup. See
  `authentication/jobox-ai-authentication.yml`.
- Most operations take a `User-Id` header identifying the acting user. It is spelled
  inconsistently across the contract (`User-Id`, `User-id`, `user-id`); match the spelling
  on the operation you are calling.
- Bodies and responses are `application/json`.

## Steps

1. **(Optional) Parse a free-text job description.** `POST /jobs/description/parse`
   (`create_9`, body `AddressParsingRequestDTO`) turns an unstructured request — the kind that
   arrives by SMS or WhatsApp — into the dispatch parameters Jobox matches on. Use this when
   you are forwarding a customer message rather than a structured form.
2. **Create the job.** `POST /jobs` (`create_8`, body `JobRequestDTO`). Jobox parses the job,
   selects a cluster of pros by skillset, availability and location, and sends it to all of
   them at once; the first to respond takes it.
3. **Read it back.** `GET /jobs/{id}` (`read_7`) or the newer `GET /v2/jobs/{job_id}`
   (`readJob`). Prefer `v2` for new integrations — `v3` exists for wallet and reports but not
   for jobs. `POST /jobs/{job_id}` (`readJobUnit`) returns the job unit.
4. **Follow progress.** `GET /jobs/{job_id}/log` (`readAll`) or `GET /v2/jobs/{job_id}/log`
   (`readAll_3`) returns the job log. `GET /jobs/{id}/description` (`readDescription`) and
   `GET /jobs/{job_id}/notes` (`readNotes`) return the narrative detail.
5. **Push a status to the customer.**
   `GET /users/{user_id}/{contact_user_id}/jobs/{job_id}/status` (`sendStatus`).

## Rules an agent must follow

- **There is no idempotency.** No `Idempotency-Key` header exists on any operation. A retried
  `POST /jobs` will create a second job. Treat job creation as at-most-once: on a timeout or
  ambiguous error, **read back** with `GET /jobs` (`read_8`) before retrying.
- **There is no dry run.** No preview, simulate or validate-only parameter exists. The first
  call is the real one.
- **Reversal is soft-delete with no stated window.** `DELETE /jobs` (`archive_3`) and
  `PUT /jobs/delete` (`archiveForAndroid`) archive a job. Jobox publishes no window inside
  which this works and no statement of what archiving does to an in-flight dispatch or an
  accepted job. Do not assume it recalls a job a pro has already taken.
- **There is no rate-limit signal.** No `RateLimit-*`, `X-RateLimit-*` or `Retry-After` header
  is returned. Back off conservatively on your own schedule.
- **Two contract paths are dead.** `GET /dashboard` and `GET /search` are published in the
  contract but return 404 on the live host. Do not build on them.
- **Correlate failures with `debug_id`.** Authorization errors carry a `debug_id` (`dbg_...`);
  quote it to support. Responses also carry `x-trace-id` and a W3C `traceparent`.
