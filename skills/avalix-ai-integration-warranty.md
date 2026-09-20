---
name: avalix-ai-integration-warranty
description: Buy Autonoma's signed seven-day availability warranty for one public HTTPS endpoint you are authorized to cover, then let anyone verify the warranty and its latest evidence without a token.
api: openapi/avalix-ai-openapi.json
operations:
  - integration_warranty_7d
  - monitor
method: generated
generated: '2026-09-19'
grounding: >-
  integration_warranty_7d and monitor exist verbatim in openapi/avalix-ai-openapi.json; GET
  /v1/warranties/{warranty_id} is declared without an operationId and named by method + path. The product
  wording ("15-minute public-endpoint checks", "approval-gated repair path", $49) is quoted from
  https://avalix.ai/autonoma/services and the offer catalog at GET /v1/offers.
---

# A signed seven-day warranty on one endpoint

The 7-Day Agent Integration Warranty is an "evidence-backed availability warranty for one authorized public HTTPS endpoint, with recurring checks, incident detection, and approval-gated repair" — $49 USDC (offer slug `integration-warranty-7d`). It is the one Autonoma product whose output is a public, verifiable record.

## 1. Confirm scope and price

`GET /v1/offers` lists `integration-warranty-7d` with `price_usdc: 49.0` and `status: active`. The services page describes it as "a signed acceptance obligation, 15-minute public-endpoint checks, incident evidence, and an approval-gated repair path". The endpoint must be public HTTPS and yours to cover — "Security work requires owner authorization and exact scope."

## 2. Activate through x402

`POST /v1/integration-warranty-7d` (`integration_warranty_7d`) with the endpoint and authorization in the JSON body. Expect `402` with the `Payment-Required` header and the PaymentRequired body (exact USDC amount, contract `0x8335…2913` on chain 8453, recipient `0xb747…0F3`, `payment_uri`). Pay exactly that, retry the same POST with `Payment-Signature`, and receive `202 "Paid job accepted"`. `400` is "Invalid input or payment proof".

For monitoring without the warranty obligation, `POST /v1/monitor` (`monitor`) follows the same 402 -> pay -> 202 pattern and returns "a bounded public-source monitoring result"; the human-facing 30-day monitoring setup is $199 and quoted separately.

## 3. Verify — no token required

`GET /v1/warranties/{warranty_id}` returns "a public signed integration warranty and its latest evidence" (`404 "Unknown warranty"` otherwise). Share the id: a third party can read the same record. The signing keys for Autonoma's signed artifacts are declared at `GET /v1/trust/jwks.json`; on 2026-09-19 that route answered `503 {"error": "passport verification key unavailable"}`, so check it before promising offline verification.

## 4. What happens on an incident

The repair path is "approval-gated": Autonoma detects and evidences the incident; a repair is proposed, not applied, until you approve it (private project thread). Any repair is quoted as separate work.

## Rules the contract states

- Seven days is the product's own bound; no early termination, extension or refund is documented.
- No idempotency key: keep the `job_id` from the 202 and the `warranty_id` it produces; do not re-POST after a timeout without checking `GET /v1/jobs/{job_id}` first.
- Never include credentials for the covered endpoint — the check is of a public HTTPS surface.
