---
name: Verify Verse's prediction track record before paying
description: Read Verse's free, Ed25519-signed calibration data and check it against the public key in its agent card, so an agent can decide whether the forecaster is worth paying before any USDC moves.
api: a2a/verse-me-com-agent-card.json
operations:
  - GET /expertise/calibration
  - GET /expertise/calibration/signed
  - GET /expertise/calibration/history/signed
  - GET /reputation
generated: '2026-09-19'
method: generated
grounding: Every path, field and key below is taken from the agent card at https://api.verse-me.com/.well-known/agent.json; nothing is invented. None of these calls was exercised live (see caveat).
---

# Verify Verse's track record

Verse sells forecasts. Its card publishes the tools to audit the forecaster for free, and an agent should use them first.

1. **Fetch the card** — `GET https://api.verse-me.com/.well-known/agent.json`. Keep `capabilities.extensions[2].params.proofOfExpertise.publicKey` (`kk+7GHTZCZX+WflypoUtQgyKGe/cWyX3czsQOsjHITs=`, Ed25519) and `keyFingerprint` (`153b303b701f1a67`). The card also carries a live `reputation` block (score, Brier score, accuracy_pct, total_predictions, resolved_predictions) — treat it as a self-report until step 3 confirms it.
2. **Read the aggregate** — `GET https://api.verse-me.com/expertise/calibration` (no auth; AGP policy `requiresAuth: false`, `dataScope: aggregate_only`). Expect Brier score, a calibration curve by confidence bucket and a domain breakdown.
3. **Read the signed copy and verify it** — `GET https://api.verse-me.com/expertise/calibration/signed`. The card states the signature method: Ed25519 over the canonical JSON serialisation (sorted keys, no whitespace). Canonicalise the payload the same way and verify against the key from step 1. A mismatch means the data did not come from the key the card advertises.
4. **Check the trend** — `GET https://api.verse-me.com/expertise/calibration/history/signed` for daily snapshots (Brier trajectory, prediction volume, resolution velocity) and `GET /reputation` for the proof bundle.
5. **Cross-check identity on-chain (optional)** — the card binds itself to ERC-8004 agentId `29481` in registry `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432` on Base. `ownerOf(29481)` should equal the x402 `payTo` wallet in the card (`0x1060D0B596685Ff429BD64f41611f5D1d15f1659`); API Evangelist confirmed this on 2026-09-19.

## Rules

- These endpoints are declared free and public; do not attach a payment.
- The card's `reputation` block is the provider's own number. Only a signature that verifies against the published key makes it evidence.
- No rate limits are published (rate-limits/verse-me-com-rate-limits.yml); the card is cacheable for an hour (`cache-control: public, max-age=3600`).

## Caveat

On 2026-09-19 every application path on api.verse-me.com answered a Cloudflare managed challenge (HTTP 403) to a non-browser client; only the card itself was served. If step 2 returns HTML titled "Just a moment...", the edge has refused the client, not the API — stop, do not attempt to evade the challenge, and contact x402@verse-me.com.
