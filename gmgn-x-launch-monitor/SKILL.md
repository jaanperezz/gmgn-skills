---
name: gmgn-x-launch-monitor
description: Watch a short list of X accounts (free: FxTwitter polling every 4 s per account + 6551 OpenTwitter real-time push) and match what they say — $TICKERs, capitalized names, posted contract addresses — against pump.fun launches within ±30 minutes, then hand the match to GMGN for security checks and (optionally) a TP/SL order. Use to turn "someone big just said X" into a measured launch signal instead of a guess.
---

# gmgn-x-launch-monitor

## Why a short list and not the firehose
X's filtered stream is Enterprise-only; Nitter is dead (C&D Aug-2026). A list of 20–50 accounts polled every 4 s each via FxTwitter (free, no key) plus the 6551 OpenTwitter WebSocket (free token at app.newsliquid.com/mcp) gives push latency on the accounts that matter, at 0 €.

## Pipeline
1. **Ingest**: own posts only (no RTs, no replies). Extract `$TICKER`, Solana CAs (base58 32–44), capitalized terms ≥4 letters minus stopwords, URLs.
2. **Launch feed**: `wss://pumpportal.fun/api/data` `subscribeNewToken` → mint, name, symbol, creator, IPFS metadata (twitter, website).
3. **Match**:
   - `x_ca`: an account posted a CA → signal at first DexScreener price.
   - `x_buzz_launch`: a term said by ≥2 accounts within 15 min AND a launch whose symbol/name contains it within ±30 min.
   - `oficial`: launch metadata points to the account that spoke (see `gmgn-official-token-check`).
4. **GMGN gate**: `token_security` + `token_info` vetoes (honeypot, rug_ratio > 0.3, bundlers > 0.3, liquidity < 5k$).
5. **Act**: paper by default (recorded in `gmgn-signal-bank`); with a verified rule and an explicit human go-ahead, `/v1/trade/swap` with `condition_orders` (TP/SL/trailing) in the same request.

## 6551 events used
`NEW_TWEET`, `NEW_TWEET_QUOTE`, `CA`, `CA_CREATE`, `UPDATE_NAME`, `UPDATE_DESCRIPTION` (renames before a CA = red flag), `NEW_FOLLOWER` (who the account just followed).

## Setup
Two text lists: `x_handles.txt` (one handle per line, 20-50 accounts) and `marcas.txt` (`name|handle|domain`). Env: `OPENNEWS_TOKEN` (optional, enables the 6551 push). Reference implementation in Python (asyncio: FxTwitter poller, 6551 and pump.fun websockets, DexScreener marker), private for now; the matching rules above are the whole logic.

## Caveats
Insiders buy before the post (LIBRA/MELANIA); hacked accounts post CAs. The signal is measured, not trusted.
