---
name: gmgn-signal-bank
description: Paper-first measurement bank for GMGN signals. Before risking money, record every candidate signal (KOL buys, smart-money buys, token_signal types, callouts, X-posted CAs, official-brand launches) with its entry price, mark it at +5/+15/+60 min/+6 h/+24 h, compare against a random same-hour control token, and report the NET return after GMGN fees. Use when an agent wants to know "does this signal actually pay?" instead of "fire on this signal".
---

# gmgn-signal-bank

Every GMGN skill on the market *fires*. None of them *measures*. This skill is the measuring stick.

## What it does
1. **Pre-registers rules** in a JSON file (`rules_g.json`): the trigger, the thresholds, the horizons, the fee model. The file is never edited after the first signal; a new idea is a new version. This is what stops you from fooling yourself.
2. **Collects signals** from GMGN OpenAPI (`/v1/user/kol`, `/v1/user/smartmoney`, `/v1/market/token_signal`, `/v1/trenches`, `/v1/callout/global`) and free sources (X via FxTwitter / 6551 OpenTwitter WS, pump.fun WS, DexScreener), one row per `(rule, source, mint)`.
3. **Pairs each signal with a control**: a random token from `trenches.new_creation` created in the same 10-minute window and not signalled by anything. Same marks, same costs.
4. **Marks** every signal at +300/+900/+3600/+21600/+86400 s with DexScreener price and liquidity.
5. **Reports NET return**: 1 % GMGN fee per leg + priority/tip per leg + constant-product impact on half the pool + 1 % base slippage, at your position size. Verdict per rule only when n ≥ 30: median and % positive vs its control.

## How it runs
Reference implementation: a Python collector (asyncio, one task per source) writing to SQLite, a marker that fills the +5/+15/+60 min/+6 h/+24 h marks, and a reporter that prints one row per rule against its control. Runs as a systemd service.
Config: `rules_g.example.json` (next to this file: the pre-registered rules, the cost model and the horizons), plus two text lists (X handles; brands as `name|handle|domain`).
Works with GMGN's public demo key (read-only) or your own key in `GMGN_API_KEY`. The collector code is private for now; the rule file and the procedure above are the whole method.

## Output (example columns)
`regla · n · n@900s · mediana_neta · media_neta · %>0 · mejor · peor · control_mediana · veredicto`

## Rate budget
Reads only: kol 1, smartmoney 2, token_signal 1, trenches 2, callout 3 (weights). Fits in the Free plan (5/s) with room to spare.

## Chains
SOL (tested). BSC/Base: same endpoints, change `chain`.

## Honest caveats
- Entry price is the DexScreener price at detection, plus the recorded polling lag (`meta.lag_s`). Real fills are worse.
- Survivorship: KOL/smart-money labels are assigned after they won. The control exists for exactly this reason.
