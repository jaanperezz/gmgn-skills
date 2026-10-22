---
name: gmgn-official-token-check
description: Decide whether a newly created token is the OFFICIAL coin of a brand, site or account, or an opportunistic copy. Cross-checks the token's IPFS metadata (twitter/website), GMGN token_info `link.twitter_username` and `dev` block (creator history, CTO flag, fund source), who actually posted the contract address on X, and `token_signal` Cto/PumpClaims. Use before sniping "the official <brand> token".
---

# gmgn-official-token-check

"Official" launches are where the money is made and where the copies are made faster. This skill grades a mint in seconds.

## Inputs
`mint` (Solana), optional `brand` (name | X handle | domain).

## Checks (each adds to the grade)
| # | Check | Source | Weight |
|---|---|---|---|
| 1 | Token metadata `twitter` resolves to the brand's handle (exact, case-insensitive), `website` resolves to the brand's domain | IPFS `uri` from pump.fun / GMGN `token_info.link` | strong |
| 2 | The CA was posted by the brand's own X account (own post, not reply/RT), within 30 min of creation | X (FxTwitter / 6551 WS `CA` event) | strongest |
| 3 | GMGN `token_info.dev`: `creator_open_count`, `creator_token_status`, `fund_from`, `cto_flag`; a creator with many previous launches is a serial copier | GMGN `/v1/token/info` | medium (negative) |
| 4 | `token_signal` type 11 (Cto) or 18 (PumpClaims) for the mint | GMGN `/v1/market/token_signal` | medium |
| 5 | Security: `rug_ratio`, `bundler_rate`, `insider_rate`, `top_10_holder_rate`, `is_wash_trading` | GMGN `/v1/token/security`, `/v1/trenches` fields | veto |
| 6 | Same symbol/name created by ≥3 different creators in the last hour (copy storm) | pump.fun WS history | negative |

## Output
```
grado: fuerte | medio | debil | copia
motivo: [lista de checks superados / fallados]
riesgo: [vetos de seguridad]
```
`fuerte` = check 2, or check 1 + no vetoes. `medio` = check 1 or 4 only. `copia` = check 6 or dev serial launcher.

## Procedure for the agent
1. `GET /v1/token/info?chain=sol&address=<mint>`: read `link.twitter_username`, `link.website`, the `dev` block and the metadata `uri`; fetch that IPFS JSON for `twitter` / `website`.
2. `GET /v1/token/security?chain=sol&address=<mint>`: the vetoes.
3. `POST /v1/market/token_signal` filtered to the mint: types 11 (Cto) and 18 (PumpClaims).
4. Brand side: FxTwitter `api.fxtwitter.com/2/profile/<handle>/statuses` (own posts only) or the 6551 `CA` event: did the brand itself post this CA, and when?
5. pump.fun websocket history: same symbol/name from other creators in the last hour.
6. Grade with the table above.
Rate budget: token_info 1 + security 1 + token_signal 1 = 3 weight per check.

## Caveat
A brand's hacked account posting a CA passes check 2. Pair with `UPDATE_NAME`/`UPDATE_DESCRIPTION` events from the 6551 WS: a profile rename minutes before the CA is a red flag.
