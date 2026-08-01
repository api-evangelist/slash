---
name: Move money between Slash accounts
description: Safely execute an intra-Slash book transfer or a virtual-account transfer, using idempotency keys so retries never double-spend.
api: openapi/slash-openapi-original.json
operations:
  - GET /legal-entity
  - GET /account
  - GET /account/{accountId}/balance
  - POST /transfers/book-transfer
  - POST /transfer/virtual-account
  - GET /transaction/{transactionId}
---

# Move money between Slash accounts

Slash's public API is in Beta and authenticates with an `X-API-Key`. Money-movement
endpoints REQUIRE an idempotency key — this skill wires that in so a retried call
returns the original result instead of moving funds twice.

## Auth
- Send `X-API-Key: <key>` on every request against `https://api.slash.com`.
- If the key is **user-scoped**, also send `x-legal-entity: <id>`. Discover valid
  ids with `GET /legal-entity` (the one call exempt from the header).

## Steps
1. `GET /account` — list accounts you can move funds between; note the source and
   destination ids (book transfers reference account/group ids like `sa_group_...`
   and `sub_...`).
2. `GET /account/{accountId}/balance` — confirm the source has sufficient balance.
3. Generate a fresh UUID and send it as `X-Idempotency-Key`.
4. Move the money:
   - `POST /transfers/book-transfer` for an instant intra-Slash transfer between two
     accounts you have access to (body: `from`, `to`, `amountCents`).
   - `POST /transfer/virtual-account` to move between virtual accounts or fund a
     virtual account from a primary account.
5. `GET /transaction/{transactionId}` — verify the resulting transaction settled.

## Rules
- Replaying the same `X-Idempotency-Key` with the SAME body returns the original
  result; the SAME key with a DIFFERENT body returns `409 Conflict`. Use one fresh
  UUID per logical transfer, and reuse it verbatim on retries.
- Errors return the custom envelope `{ success, message, identifier, rawStatus,
  displayType }` — read `message` (it carries a bracketed trace id) and `rawStatus`.
- A missing `x-legal-entity` on a user-scoped key returns `400`; no permission role
  on the entity returns `403`.
