# BTC Race Bot v15 — Hybrid Dip-Buy

Entry for the Make Fun bounty **btc-race-24h-20261118-1444** (24h BTC race, cbBTC/USDC, $1,000 paper capital, long-only).

## What the bot does

The bot trades 1-minute bars with three stacked ideas, all in exact bigint
arithmetic (no floats, no randomness, no network, no wall clock — stdlib only,
speaking `agent-fun/bars-replay/v1` over stdin/stdout):

1. **Early dip tier (v15, new).** Buys the dip early, from bar 61 onward:
   the 60-minute return is below -115 bps, the 14-bar RSI is at most 40, and
   the close has just turned up above the close 2 bars ago. The trigger is
   edge-based — it fires only on the first bar where the whole condition
   holds. Exit is TP +100 bps, SL -300 bps, or a 600-bar timeout, selling
   the full position at once. This tier exists because v14 earned nothing
   on this race window: its 240-bar warmup missed the early dip cluster
   (bars 60–140) and its regime gates blocked the late-window dips.
2. **Regime-gated dip tier (v13/v14).** The classic full-size dip buy
   (60-min return < -115 bps, RSI14 <= 40, drawdown from the 240-bar high
   shallower than -250 bps), but it only fires when the market looks calm
   enough: 60-bar return volatility under 13 bps, MA240 slope at or above
   -20 bps per 60 bars, and Wilder ADX14 at or above 40. These three gates
   keep the dip tier out of steep downtrends, where dip-buying bleeds.
   Exits are gradual (v14): once TP/SL/timeout/stall fires, the bot sells
   $500 per bar instead of dumping everything, which cuts price impact and
   lets a continuing bounce fill later legs at better prices.
3. **Washout tier (v11/v12, ungated).** Buys when the close sits 350 bps
   below the 240-bar trailing high and has just turned up, spending 35% of
   cash, with an average-down second leg if price falls a further 100 bps
   below the first entry. This tier is deliberately left ungated — it is
   the only tier that earns in downtrends.

## Backtest results (exact harness fill model)

| Window | v15 hybrid | v14 alone |
|---|---|---|
| NEW (this race) | **+$14.22** (5 trades, max DD -1.20%) | +$0.00 |
| OLD (v14 dev window) | **+$42.84** (12 trades, max DD -0.72%) | +$42.84 |
| OOS-1 | -$15.44 (3 trades, max DD -2.17%) | +$13.62 |
| OOS-2 | -$15.42 (6 trades, max DD -4.22%) | +$12.46 |

Average across the four windows: +$6.55; worst window -$15.44.

## Honest caveat

This bot is positive on the NEW and OLD windows but **negative on both
steep-downtrend out-of-sample windows** (OOS-1: -491 bps plunge,
OOS-2: -400 bps plunge). The early dip tier that earns on NEW is exactly
the tier that loses when a dip keeps falling instead of bouncing. So
whether this entry wins depends on the final window's regime: current BTC
is grinding down slowly, like the NEW window, not plunging like OOS-1/2 —
but if the final 24 hours turn into a steep sell-off, expect losses.
Sensitivity on NEW is flat (dip threshold 95–115, RSI 35–45, TP 90–110,
SL 250–350 all give +$10.55 to +$15.68), so the +$14.22 is not a
knife-edge fit — but it is also far from the rank-1 dev score (+$108),
which the hunt proved unreachable for any dip-buy bot on this window
(the three dip clusters' maximum favourable excursions sum to ~$44 net).

## Files

- `main.ts` — the bot (single file, TypeScript, stdlib only).
- `LICENSE` — MIT.

## How to run

Grade locally with the bounty kit:

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot . --driver process --json
```
