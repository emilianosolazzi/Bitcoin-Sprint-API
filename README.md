# NativeBTC Block Template Optimizer

**More fees. Every block.**

Free block template optimization API for Bitcoin mining pools. No hashpower changes. No node changes. One API call.
Try it free, verify the results yourself, then we can talk about what a deeper relationship looks like.”

---

## What it does

Bitcoin Core selects transactions by fee-rate only. It misses CPFP packages — low-rate parents paired with high-rate children — leaving fees behind every block.

This engine computes effective package fee-rates across ancestor sets and selects by package rate rather than individual rate. The result: more transactions, more fees, same block weight budget.

## Live results

Measured on live Bitcoin Core 29.0 mainnet across a 72-cycle, 6-hour shadow run:

| Metric | Value |
|---|---|
| Uplift min | 124.10 bps |
| Uplift median | 577.51 bps |
| Uplift mean | 627.06 bps |
| Uplift p95 | 1,376.18 bps |
| Uplift max | 1,499.18 bps |
| Total extra sats (6h) | 7,400,254 |
| Verifier results | ok=73 fail=0 |

Every result is Ed25519 signed. Verify independently:
→ [github.com/emilianosolazzi/Bitcoin-Template-Benchmark](https://github.com/emilianosolazzi/Bitcoin-Template-Benchmark)

---

## Quick start

```bash
# Linux / Mac
curl -X POST https://nativebtc.org/v1/template/optimize \
  -H "Content-Type: application/json" \
  -d '{"use_optimizer": true}'
```

```powershell
# Windows PowerShell
Invoke-RestMethod -Uri "https://nativebtc.org/v1/template/optimize" `
  -Method POST -ContentType "application/json" `
  -Body '{"use_optimizer": true}'
```

---

## Response

```json
{
  "template": {
    "height": 950579,
    "transactions": [ "... optimized tx list ..." ],
    "coinbase_value": 313596732
  },
  "baseline": {
    "tx_count": 1458,
    "total_fees": 1016736
  },
  "optimized": {
    "tx_count": 1689,
    "total_fees": 1096732
  },
  "uplift_bps": 786.79,
  "signed_receipt": {
    "payload": { "..." : "..." },
    "signature": {
      "alg": "Ed25519",
      "canonicalization": "json_sorted_keys_compact",
      "key_id": "08f55b7f4f76bd21",
      "public_key": "f366fae1be2c9871ac0cc3cb0f054cdfecb7d510cc722893cc108bcebb94fbc6",
      "value": "ecba259e..."
    }
  }
}
```

The `template` field is Bitcoin Core GBT-compatible. Feed it directly into your Stratum server.

---

## Request parameters

| Field | Default | Description |
|---|---|---|
| `use_optimizer` | `true` | Set to `false` for Core baseline only |
| `min_fee_rate_sat_vb` | `1.0` | Minimum fee rate filter |
| `weight_limit` | `3996000` | Target block weight |
| `max_tx_count` | `4000` | Maximum transactions |

---

## Free tier limits

| Limit | Value |
|---|---|
| Requests per IP | 1 per 15 minutes |
| Mempool size cap | 5,000 transactions |
| Signed receipts | Every response |
| Cost | Free |

---

## Verify the signed receipt

Every response includes an Ed25519 signed receipt. Verify it yourself:

```bash
# Python
python python/verify_bench_receipt.py envelope.json

# Rust  
./verify_bench_receipt envelope.json

# Over HTTP
curl -X POST https://nativebtc.org/v1/verify-receipt \
  -H "Content-Type: application/json" \
  -d '{"envelope": <receipt>, "key_hex": "<pubkey>"}'
```

Public key: `f366fae1be2c9871ac0cc3cb0f054cdfecb7d510cc722893cc108bcebb94fbc6`

---

## What the uplift means in dollars

At current prices (~$100,000 BTC, ~0.3 BTC avg fees per block):

| Your hashrate | Expected blocks/year | Engine value/year |
|---|---|---|
| 1,000 TH/s | ~0.06 | ~$1,000 |
| 10,000 TH/s | ~0.6 | ~$10,000 |
| 100,000 TH/s | ~6 | ~$100,000 |
| 1 EH/s | ~65 | ~$1,100,000 |
| 10 EH/s | ~650 | ~$11,000,000 |

*Based on median 578 bps uplift on 0.3 BTC fees at $100k BTC price.*

---

## How it works (high level)

Standard Core selection: sort all transactions by fee-rate, fill block greedily.

This engine: compute effective fee-rate across each transaction's full ancestor package. Select by package rate. Capture CPFP relationships Core's greedy pass misses.

Result: 411+ extra transactions per block on congested mempools. Never worse than Core baseline.

---

## Proof artifacts

Full 72-cycle signed benchmark:
→ [github.com/emilianosolazzi/Bitcoin-Template-Benchmark](https://github.com/emilianosolazzi/Bitcoin-Template-Benchmark)

Contents:
- `manifest.json` — signed manifest pinning all 72 cycle envelopes by SHA-256
- `summary.md` — run parameters, results, verification commands
- `pubkey.hex` — Ed25519 public key for independent verification

---

## Integration guide

Full documentation: [nativebtc.org/docs](https://nativebtc.org/docs)

---

## Contact

Emiliano Solazzi  
emiliano@nativebtc.org

---

*© 2026 Emiliano Solazzi. All rights reserved. Source code not licensed for use.*
