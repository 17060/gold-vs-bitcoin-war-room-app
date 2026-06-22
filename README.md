# Gold vs Bitcoin War Room

A single-screen regime dashboard that tells you, at a glance, whether the day is
**Fear**, **Liquidity**, or **Store-of-Value** driven.

**Live site:** https://17060.github.io/gold-vs-bitcoin-war-room-app/

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

## Intelligence + distribution (v4)
- **Backtest hit-rate** badge — how often the regime call matched next-day direction over the trailing window.
- **Regime-conditional stats** — average next-day BTC/gold return on past days of the current regime.
- **Divergence flags** — auto-callout when signals conflict (e.g. Crypto F&G vs vol gauge).
- **Anomaly z-scores** — toasts when a move is ≥2σ vs its 20-day norm.
- **More inputs** — equity proxy (SPY/VIX), real yields (TIP), 4th stablecoin (FRAX).
- **Scenario sandbox** — sliders for BTC/gold/DXY/10Y/peg that recompute the regime + mix live.
- **Annotations/journal** — pin notes to dates; markers show on the 30-day calendar (session-scoped).
- **Custom alert thresholds** — tune peg/big-move/ratio triggers in the UI.
- **Discord webhook** — alerts post to a channel if `data.discord_webhook` is set.
- **PWA** — installable (manifest + icons); **OG image** for link previews; **keyboard nav + ARIA** on tiles.

## Decision layer (v3)
- **Playbook line** per regime — suggested posture.
- **Confidence meter** — decisiveness from the top-two bucket gap + activity.
- **Streak** — how long the current regime has held (from the intraday timeline).
- **Tripwires** — live "what would flip it" thresholds.
- **Multi-timeframe regime** — 1D / 1W / 1M verdicts side by side (daily closes).
- **30-day regime calendar** — daily dominant-regime heatmap.
- **Real Crypto Fear & Greed** (alternative.me) alongside the vol-derived gauge — divergence is a signal.
- **Dollar-adjusted moves** — BTC/gold net of DXY.
- **Yield curve** — 10Y minus 13-wk T-bill.
- **GLD flow proxy** — volume vs average.
- **Session tag**, **per-feed freshness dots**, **sanity banner** (flags non-reconciling data), **drill-down slide-over** per tile, **methodology popovers**, **PNG snapshot export**.

## Features
- **Live-computed regime** verdict + mix bar (fear / liquidity / store-of-value).
- **"Why this verdict"** expander — per-instrument contribution to the score.
- **BTC/Gold ratio tile** (oz per BTC) with who's-leading tag + sparkline.
- **Fear & Greed gauge** derived from BTC realized volatility.
- **BTC–Gold correlation** readout (hourly-return correlation).
- **Regime-today timeline** — intraday verdict per hour as a colored strip.
- **Stablecoin peg board** — USDT, USDC, DAI + total tracked cap.
- **Real sparklines** for BTC, gold, DXY (10Y omitted — stale Fri close).
- **Data-aware LIVE badge** — shows PARTIAL + stale-feed count when any feed is old.
- **Controls:** 1D/1W/1M timeframe, CSV export, light/dark toggle, manual refresh.
- **Auto-refresh** every 2 min (re-fetches `data.json` and re-classifies).
- **Alerts** (in-page toasts): regime flip, peg break (>25 bps), big move (>3%). Thresholds in `data.alerts`.

## Refresh
The page auto-refreshes `data.json` every 2 minutes. To update the underlying numbers,
re-pull quotes + macro + hourly histories, regenerate `data.json`, and redeploy. A
scheduled task can do this automatically — see the cron set up in the project thread.
