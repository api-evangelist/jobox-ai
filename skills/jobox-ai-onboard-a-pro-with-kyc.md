---
name: jobox-ai-onboard-a-pro-with-kyc
description: Create a Jobox user, run the KYC identification upload, attach bank details, and get the pro ready to be paid. Jobox handles KYC on behalf of demand partners.
api: Jobox Kili API
base_url: https://api.jobox.ai/Kili
operations:
  - create_17
  - read_16
  - update_6
  - getInfo
  - submitInfo
  - getKycUploadUrl
  - uploadId
  - read_12
  - create_1
  - update
  - readByUserId
  - archive
  - readSummary
generated: '2026-08-23'
method: generated
source: openapi/jobox-ai-kili-openapi.json + https://www.jobox.ai/marketplace
---

# Onboard a pro and clear KYC on Jobox

Jobox's marketplace pitch is that it handles KYC for you. These are the operations that do it.

## Steps

1. **Create the user.** `POST /users` (`create_17`, body `User`). Read back with
   `GET /users/{user_id}` (`read_16`).
2. **Update the user.** `PUT /users/{user_id}` (`update_6`).
   **Do not use `PUT /users`** — its operationId is literally `updateDeprecated`, the only
   deprecation marker anywhere in this contract.
3. **Check KYC state.** `GET /users/{user_id}/kyc_info` (`getInfo`) returns `KycInfoDTO`.
4. **Submit KYC.** `POST /users/{user_id}/kyc_info` (`submitInfo`, body `KycInfoDTO`).
5. **Upload identification.** Get a signed upload target with
   `GET /upload/{user_id}/kyc` (`getKycUploadUrl`), then `POST /upload/identification`
   (`uploadId`, `multipart/form-data`). Verify with `GET /identification/{user_id}` (`read_12`).
6. **Attach the bank account.** `POST /bankinfo` (`create_1`, body `BankInfo`); amend with
   `PUT /bankinfo` (`update`); read with `GET /bankinfo/users/{user_id}` (`readByUserId`).
7. **Set the trade.** `GET /occupations` (`readSummary`) lists the occupations Jobox matches on.

## Rules an agent must follow

- **This flow moves personally identifiable information and government identification.** Do not
  cache, log or echo the contents of `KycInfoDTO` or an identification upload. The contract
  carries no field-level descriptions, so treat every field in these bodies as sensitive.
- **One operation takes a `password` header parameter** (`PUT /users/{user_id}/{verification_code}`).
  Never place a credential in a URL or a log line.
- **Reversal:** `DELETE /bankinfo` (`archive`) detaches a bank account. There is no published
  window and no documented undo for a submitted KYC record. Jobox states no retention or
  deletion policy for identification documents in anything it publishes.
- No idempotency key: a retried `POST /users` may create a duplicate pro. Read back first.
- Anonymous calls return 401; the credential is undocumented and sales-gated.
