---
name: avalix-ai-free-check-then-paid-lint
description: Rehearse an A2A agent card, MCP manifest or OpenAPI document against Autonoma's free deterministic checks, then buy the paid deterministic lint or repair through x402 (HTTP 402 -> pay exact Base USDC -> retry with Payment-Signature) and read the result from the job.
api: openapi/avalix-ai-openapi.json
operations:
  - agent_card_lint
  - mcp_compatibility
  - openapi_repair
  - receipt_verifier
method: generated
generated: '2026-09-19'
grounding: >-
  Every operationId above exists verbatim in openapi/avalix-ai-openapi.json. Routes the provider declares
  without an operationId (POST /v1/trust/preview, GET /v1/jobs/{job_id}, GET /v1/samples/*) are named by
  method + path. The 402 shape is the one observed live on POST /v1/receipt-verifier on 2026-09-19; the
  payment rules are quoted from https://avalix.ai/llms.txt. Autonoma publishes no skills or AGENTS.md of its
  own (probed 2026-09-19).
---

# Check for free, then pay for the lint

Autonoma sells deterministic checks of the same documents this catalog profiles — A2A agent cards, MCP manifests, OpenAPI files, action receipts. Every paid check has a free rehearsal. Use it first.

## 1. Rehearse on the free surface (no wallet, no account)

- `POST /v1/trust/preview` with `{"agent_card": {…}, …}` — a free, non-persistent Trust Preview of a supplied card, permissions and authorization boundary. An empty body is rejected with `400 {"error": "agent_card must be a JSON object"}`. The provider tags this skill `rate-limited` without publishing the threshold; do not loop on it.
- MCP tool `compatibility_check` at `https://avalix.ai/autonoma/mcp` with `{"kind": "a2a" | "mcp" | "openapi" | "auto", "url": "https://…"}` or `{"document": {…}}` — static checks with evidence hashes and explicit limitations. Same check on the web at https://avalix.ai/autonoma/check.
- Read the output shape before buying: `GET /v1/samples/agent-card-lint` and `GET /v1/samples/receipt-verifier` return `sample: true` reports with `summary`, `checks[]`, `findings[]` and `limitations[]`.

## 2. Pick the paid twin and read its price

`GET /v1/offers` returns the eight machine offers with `price_usdc` (mcp-compatibility $3, openapi-repair $5, agent-contract $5, receipt-verifier $1, …). The card-lint product is $1.00 and the data validator $0.10 via the `/x402/*` relay routes listed in https://avalix.ai/autonoma/.well-known/x402-services.json.

## 3. POST once to receive the challenge

`POST /v1/agent-card-lint` (`agent_card_lint`), `POST /v1/mcp-compatibility` (`mcp_compatibility`), `POST /v1/openapi-repair` (`openapi_repair`) or `POST /v1/receipt-verifier` (`receipt_verifier`) with the document as the JSON body. Expect **HTTP 402** with:

- a `Payment-Required` header, and
- a JSON body `{"status": "payment_required", "quoted_usdc": 1.0, "network": "base", "chain_id": 8453, "asset": "USDC", "asset_contract": "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913", "recipient": "0xb747D079416A84d7F35e686Ea4a4252aacBEA0F3", "payment_uri": "ethereum:0x8335…@8453/transfer?address=0xb747…&uint256=1000000", "submit": "…"}`.

Treat the live challenge as authoritative for amount, contract and recipient. Do not pay an amount or recipient you read anywhere else.

## 4. Pay and retry the same POST

From llms.txt: "Use an x402-compatible wallet to create the signed Payment-Signature authorization, then retry the same POST. A Payment-Signature is not a pasted transaction hash." The observed 402 body's `submit` line says `Payment-Signature: <Base transaction hash>` — follow whichever the live challenge you received states. The x402 manifest's `maxTimeoutSeconds` is 60. A retry that the server cannot verify returns `400 "Invalid input or payment proof"`.

## 5. Read the result

The paid route answers **202 "Paid job accepted"**. Poll `GET /v1/jobs/{job_id}` (uuid) with the report token in `Authorization: Bearer …`; `401` means a missing or wrong token, `404` an unknown job. There are no webhooks (agent card `pushNotifications: false`).

## Rules the contract states

- Never send a private key, seed phrase, password, or unrelated confidential data. Public descriptor metadata only.
- No idempotency key exists on any paid route; a retry after an ambiguous 202 may be a second purchase. Keep the `job_id` from the first 202 you receive.
- Results are "bounded evidence, not certification or a guarantee of security" (info.description).
- Refunds for completed jobs are not offered by API; the site Terms say to email within 14 days and the /autonoma/ page says delivered work is not refundable.
