---
name: Book a Curri delivery
description: Quote and book a last-mile delivery through the Curri GraphQL API, then track it to completion.
api: https://api.curri.com/graphql
operations: [deliveryQuote, bookDelivery, delivery]
---

# Book a Curri delivery

Operating instructions for an agent using Curri's GraphQL API to price, book, and
track a last-mile construction/industrial delivery. Ground every call in the real
documented operations below — do not invent fields.

## Authentication
- Endpoint: `https://api.curri.com/graphql`
- Auth: HTTP Basic. Send `Authorization: Basic <base64(userId:apiKey)>`.
- Use the **Sandbox key** (issued alongside the live key) while testing: no drivers
  are dispatched and no charges are processed; such deliveries carry the `test` status.

## Steps
1. **Quote (optional).** Call the `deliveryQuote` query with `origin`, `destination`,
   `manifestItems`, `deliveryMethod`, and `priority` (Rush / Same Day / Scheduled).
   A quote expires after **15 minutes**.
2. **Book.** Call the `bookDelivery` mutation. Either pass the `deliveryQuoteId`
   returned from the quote, OR pass the same parameters with `skipQuote: true`.
   Required inputs: `origin`, `destination`, `manifestItems`, `deliveryMethod`.
   Useful optionals: `pickupContact`, `dropoffContact`, `scheduledAt`, `deliveryMeta`,
   `declaredValue`, `requirements`, `attachments`.
3. **Track.** Read the returned delivery object's tracking URL, or call the `delivery`
   query by id. Poll for the `status` field, or register a webhook (see below).

## Conventions and error handling
- **Errors:** GraphQL errors come back with HTTP 200 and an `errors` array at the JSON
  root — always inspect it even on a 200. Malformed/incomplete payloads return HTTP 400.
- **No idempotency key** is supported; guard against duplicate bookings client-side.
- **Statuses** progress: `pending` → `en_route_to_origin` → `at_origin` →
  `en_route_to_destination` → `at_destination` → `delivered`; terminal states also include
  `canceled` (with `cancellationReason`).
- **Webhooks:** Curri POSTs delivery updates to a configured URL about every 20 seconds
  until a terminal status. Subscription is provisioned by the Curri team.

See also: `conventions/curri-conventions.yml`, `errors/curri-problem-types.yml`,
`asyncapi/curri-webhooks.yml`, `authentication/curri-authentication.yml`.
