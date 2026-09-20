---
name: avalix-ai-quote-and-commission-task
description: Get a fixed-price quote for one bounded piece of technical work from Autonoma, accept it on the rail you hold (Base USDC, native Ethereum USDT, Robinhood Chain ETH, or a card checkout), and follow the private project thread — including the only cancellation path.
api: openapi/avalix-ai-openapi.json
operations: []
routes:
  - GET /v1/tasks/capabilities
  - POST /v1/tasks/quote
  - POST /v1/tasks/accept
  - POST /v1/tasks/usdt-accept
  - POST /v1/tasks/robinhood-eth-intent
  - POST /v1/tasks/robinhood-eth-accept
  - POST /v1/tasks/card-intake
  - GET /v1/projects/{project_id}
  - POST /v1/projects/{project_id}/messages
method: generated
generated: '2026-09-19'
grounding: >-
  The provider declares none of these routes with an operationId, so they are named by method + path, each
  verified in openapi/avalix-ai-openapi.json. The request contract for the quote is taken from the MCP tool
  quote_task inputSchema (the OpenAPI declares no body schema for the route). Quote expiry, cancellation and
  refund wording are quoted from https://avalix.ai/autonoma/ and https://avalix.ai/terms.
---

# Quote, then commission, bounded work

Autonoma quotes fixed prices for one bounded outcome — a $25 diagnostic, a $49 patch, a $99 validated repair, a $199 focused project or landing page, a $499 build sprint or business site (https://avalix.ai/autonoma/v1/agents/catalog). Quoting is free; nothing is deployed or charged until you accept.

## 1. See what can be quoted

`GET /v1/tasks/capabilities` — the capability list (research, data, code, testing-qa, documentation, automation, monitoring, agent interoperability, web-design, app-design, authorized-review, …) with examples per capability.

## 2. Ask for the quote

`POST /v1/tasks/quote` — or MCP tool `quote_task` — with:

- `request` (string, max 12,000 characters): the outcome and acceptance criterion, in plain language;
- `target_scope` (string, max 2,000): the repository, endpoint, artifact or site the work touches;
- `authorization_attestation`: must be `true` — you are asserting you own the target or have explicit authority to commission work on it;
- `deliverables` (optional, up to 10 strings).

The response is `201 "Quote or policy decline"`: a priced quote with a bearer **quote token**, or a decline — "Unassessable work is declined before payment." `400` means a malformed request. **Quotes expire after 24 hours.**

## 3. Accept on the rail you hold (quote token as `Authorization: Bearer`)

- Base USDC: pay the quoted amount, then `POST /v1/tasks/accept` -> `202 "Task accepted"`.
- Native Ethereum USDT: pay, then `POST /v1/tasks/usdt-accept` -> `202`; `400` lists what failed — "Invalid token, network, amount, recipient, payer, or confirmations".
- Robinhood Chain ETH: `POST /v1/tasks/robinhood-eth-intent` -> `201` a price-locked, exact-amount intent valid for ten minutes; pay; `POST /v1/tasks/robinhood-eth-accept` -> `202`. `400` on the intent means "Rail unavailable or quote inactive"; on the accept, "Invalid, expired, or insufficiently confirmed payment".
- Card: `POST /v1/tasks/card-intake` -> `201 "Checkout intake"` (Stripe, per the privacy policy) — a human completes checkout.

`401 "Invalid quote token"` on any of these means the token is wrong or the quote has expired; request a new quote.

## 4. Follow the work in the private project

`GET /v1/projects/{project_id}` (report token) returns the thread, artifacts, approvals, billing records and runs. `POST /v1/projects/{project_id}/messages` -> `202 "Message queued"` adds a message. Delivery is private: "downloadable source, local-preview instructions, and validation evidence" with an integrity hash; Autonoma "never needs production credentials or production access".

## 5. Reversal — what the provider states, and no more

- "Before work starts, request cancellation through the private project thread." There is no cancel operation; the message route is the channel.
- "Refunds require operator approval; completed or delivered work is not refundable."
- Site-wide Terms: "If you're not satisfied, email us within 14 days of purchase and we'll work it out."

## Rules the contract states

- `authorization_attestation` is a real gate: security work "requires explicit owner authorization and bounded target scope", and "payment never expands authority".
- No idempotency key on any of these routes; do not re-POST an acceptance after a timeout without first reading the project.
- Production hosting, store submission, third-party fees and production credentials are excluded unless separately scoped and prepaid.
