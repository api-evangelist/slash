---
name: Issue and control a corporate card
description: Create a virtual or physical Slash card, apply a spending constraint, and monitor its utilization.
api: openapi/slash-openapi-original.json
operations:
  - GET /account
  - GET /card-product
  - POST /card
  - PUT /card/{cardId}/spending-constraint
  - GET /card/{cardId}/utilization
  - GET /card/{cardId}
---

# Issue and control a corporate card

## Auth
- `X-API-Key: <key>` against `https://api.slash.com`; add `x-legal-entity` for
  user-scoped keys.

## Steps
1. `GET /account` — choose the account (and optionally virtual account) the card
   draws from.
2. `GET /card-product` — pick a `cardProductId` if issuing a specific product.
3. `POST /card` — create the card. Supply `accountId` (and optionally
   `virtualAccountId`, `cardGroupId`, `cardProductId`, and your own `userData`
   like an internal id or email).
4. `PUT /card/{cardId}/spending-constraint` — set the full spending constraint
   (limits, merchant/category rules). Use `PATCH` on the same path to adjust
   individual properties later without replacing the whole constraint.
5. `GET /card/{cardId}/utilization` — read current spend against the constraint;
   `GET /card/{cardId}` for full card detail.

## Rules
- Card PAN/CVV are sensitive. When reading card data through the hosted MCP server,
  supply an `rsaPublicKey` so PAN/CVV come back RSA-OAEP encrypted rather than
  plaintext.
- Errors use the custom envelope `{ success, message, identifier, rawStatus,
  displayType }`.
- Card lifecycle events (`card.update`, `card.delete`, `card_creation.event`) are
  delivered via registered webhooks — fetch the entity by `entityId` on receipt.
