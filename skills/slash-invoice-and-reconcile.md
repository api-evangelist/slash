---
name: Invoice a customer and reconcile payment
description: Create a customer, issue an invoice, and reconcile it against the transactions that paid it.
api: openapi/slash-openapi-original.json
operations:
  - POST /customer
  - GET /invoice/validate-number
  - POST /invoice
  - GET /invoice/{invoiceId}
  - GET /transaction
  - POST /invoice/{invoiceId}/reconcile
  - POST /invoice/{invoiceId}/void
---

# Invoice a customer and reconcile payment

## Auth
- `X-API-Key: <key>` against `https://api.slash.com`; add `x-legal-entity` for
  user-scoped keys.

## Steps
1. `POST /customer` — create the billing recipient (or reuse an existing customer id).
2. `GET /invoice/validate-number` — confirm your intended invoice number is free.
3. `POST /invoice` — create the invoice (account, line-item details, payment methods,
   optional auto-pull).
4. `GET /invoice/{invoiceId}` — poll status until paid.
5. `GET /transaction` — locate the inbound transaction(s) that settled the invoice.
6. `POST /invoice/{invoiceId}/reconcile` — link those transactions to the invoice
   (marks it paid). Use `POST /invoice/{invoiceId}/unreconcile` to unlink, or
   `POST /invoice/{invoiceId}/void` to void an unpaid invoice.

## Rules
- For recurring billing, use the `/invoice/series` endpoints instead of one-off
  invoices; the contact cannot be changed after a series is created.
- Errors use the custom envelope `{ success, message, identifier, rawStatus,
  displayType }`.
