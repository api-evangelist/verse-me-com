---
name: Ask Verse a paid question via x402
description: Price a concrete forecasting or strategy question, decide whether to pay, then submit it through the x402 402-pay-retry flow and verify the signed answer. Money moves on-chain and is not reversible, so the price check comes first.
api: a2a/verse-me-com-agent-card.json
operations:
  - POST /expertise/consultation/price-estimate
  - POST /expertise/consultation
  - POST /expertise/epistemic-audit
  - POST /expertise/edgar
generated: '2026-09-19'
method: generated
grounding: Paths, request body, price tiers, network, asset, headers and the pay-retry sequence are taken from the agent card at https://api.verse-me.com/.well-known/agent.json (gettingStarted, skills, securitySchemes, extensions) and the CORS headers on the card response. Nothing is invented and nothing was purchased.
---

# Ask Verse a paid question

## Before spending

1. **Write one concrete question** with a timeframe or a decision to evaluate (the card's guidance). Choose a `domain` — the example is `ai-capabilities`.
2. **Get the price first** — `POST https://api.verse-me.com/expertise/consultation/price-estimate` with `{"question": "...", "domain": "..."}`. This is free and is the only rehearsal the provider offers. Published tiers: quick $3, standard $10, deep $25, research $50 (USDC). EDGAR requests (`POST /expertise/edgar`) are $0.01 index / $0.05 analysis / $0.25 deep. `strategy_recommendation` and `epistemic_audit` are paid but carry no published figure — the estimate is the only way to learn it.
3. **Apply the spend cap you were given.** There is no refund, cancel or dispute path published (conventions/verse-me-com-conventions.yml, reversibility: none). Once the USDC transfer settles on Base it is final.

## Paying and asking

4. **Submit** — `POST https://api.verse-me.com/expertise/consultation` with the same JSON body (`curl -X POST ... -H 'Content-Type: application/json' -d '{"question":"...","domain":"ai-capabilities"}'` is the card's own example). Expect **HTTP 402** carrying the payment requirements; the price quoted here is authoritative.
5. **Pay exactly the quoted amount** in USDC on Base (`eip155:8453`, USDC contract `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`) to the `payTo` address in the requirements — the card publishes `0x1060D0B596685Ff429BD64f41611f5D1d15f1659` — via the x402 flow (facilitator `https://openfacilitator.io`). Sign the authorisation with the wallet you control.
6. **Retry the identical request with the receipt** in the payment header (the edge names `PAYMENT-SIGNATURE` and `X-PAYMENT` as allowed request headers; read `PAYMENT-RESPONSE` on the reply). Do not change the question between the 402 and the retry — the payment is bound to the quote.
7. **Verify the answer** — the response carries a confidence level, domain adjustments, reasoning and an Ed25519 signature. Verify it over canonical JSON (sorted keys, no whitespace) against the card's `proofOfExpertise.publicKey`. Store the signed payload; it is your receipt.

## Rules

- No account or API key exists; do not look for one.
- No idempotency key is documented. If a retry after a network failure is unavoidable, resend the **same** payment receipt rather than paying again.
- The AGP policy marks consultation `mode: async`, while the skill text says results return immediately. If the paid reply is not the answer itself, the card documents no polling path — stop and ask x402@verse-me.com rather than paying twice.
- No rate limits are published.

## Caveat

On 2026-09-19 every application path, including the free price-estimate, answered a Cloudflare managed challenge (HTTP 403) to a non-browser client. If any step returns HTML titled "Just a moment...", the edge refused the client; do not attempt to bypass it.
