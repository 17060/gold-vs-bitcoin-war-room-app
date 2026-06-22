# Gold vs Bitcoin War Room

A single-screen regime dashboard that tells you, at a glance, whether the day is
**Fear**, **Liquidity**, or **Store-of-Value** driven.

## Files
- `index.html` — the dashboard (static HTML/CSS/JS, no build step)
- `data.json` — the live snapshot + regime read that the dashboard renders

## The normalized view
All instruments are pulled from one source (Perplexity Finance real-time quotes)
and reduced to a common shape: latest price, today's open, % change since open,
plus an intraday sparkline where available.

| Instrument | Ticker | Role in the read |
|---|---|---|
| Bitcoin | BTCUSD | Risk appetite / liquidity proxy |
| Gold | XAUUSD | Store-of-value demand |
| Dollar Index | DX-Y.NYB | Global liquidity tide (inverse) |
| 10Y Treasury | ^TNX | Rate / duration pressure |
| Tether | USDTUSD | Crypto funding stress (peg) |
| USD Coin | USDCUSD | Crypto funding stress (peg) |

Macro backdrop (Fed funds, core CPI, headline CPI) comes from Fed / BLS via the
macro snapshot.

## Regime logic (computed live)
The verdict is **derived in-page** by `classifyRegime()` in `index.html` from the live
`change_pct` of BTC, gold, DXY, and 10Y plus the stablecoin peg deviations — it is NOT
hardcoded, so it updates whenever `data.json` changes. Each instrument votes into one of
three buckets; scores are normalized to 100 and the dominant bucket wins (blended label
when the top two are within 12 points). A near-flat tape returns "Mixed / Neutral", and any
peg break (>25 bps) forces "Fear · Funding Stress".

To pin a manual verdict, add a `regime` object to `data.json` with `pinned: true` plus
`label`, `code`, `tagline`, and `score`.

- **Fear** — BTC down hard, gold bid, dollar up, yields up, and/or stablecoins
  off peg (>25 bps). Flight to cash/quality.
- **Liquidity** — BTC + gold both rising while the dollar softens and yields ease.
  A rising tide lifting hard assets. (Today's read.)
- **Store-of-Value** — gold leads, BTC mixed, dollar weak, real-asset demand
  driven more by debasement than risk appetite.

Peg health is a hard gate: if USDT or USDC breaks ~25 bps from $1.00, funding
stress pushes the read toward Fear regardless of price action.

## Snapshot (2026-06-22 10:03 UTC)
Verdict: **Liquidity / Store-of-Value** — dollar softening lifting hard assets,
pegs intact, no funding stress.

## Refresh
Re-pull quotes + macro, regenerate `data.json`, redeploy. The dashboard reads
`data.json` at load — no code changes needed to update the numbers.
