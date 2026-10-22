# GMGN skills by TaurozStudios

Three agent skills (SKILL.md format, for Claude Code / OpenClaw style agents) built on the public [GMGN OpenAPI](https://docs.gmgn.ai) plus free sources (FxTwitter, 6551 OpenTwitter, pump.fun websocket, DexScreener). Read-only by default: every skill records to paper first and only hands a trade to GMGN with an explicit human go-ahead.

| Skill | Category | What it adds |
| --- | --- | --- |
| [gmgn-signal-bank](gmgn-signal-bank/SKILL.md) | Data Analytics | Measures any signal (KOL, smart money, token_signal, callouts, X) in paper against a random control, reporting NET return after GMGN fees. Rules are pre-registered in JSON before the first signal. |
| [gmgn-official-token-check](gmgn-official-token-check/SKILL.md) | Monitor | Grades whether a new token is the OFFICIAL coin of a brand or account, or a copy: IPFS metadata, GMGN `link` and `dev` blocks, who posted the CA on X, Cto / PumpClaims signals. |
| [gmgn-x-launch-monitor](gmgn-x-launch-monitor/SKILL.md) | News & Media | Matches what a short list of X accounts say ($TICKER, names, CAs) with pump.fun launches within 30 minutes, then gates the match through GMGN security checks. |

Chains: Solana (tested); BSC / Base use the same endpoints with another `chain`.
No keys or secrets in this repo. Contact: [@taurozstudios](https://x.com/taurozstudios).
