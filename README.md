# Bradestudio-btc-race-24h-20261118

[![make.fun · btc-race-24h-20261118-1444](https://make.fun/badge/btc-race-24h-20261118-1444.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![rank](https://make.fun/badge/btc-race-24h-20261118-1444/Bradestudio/Bradestudio-btc-race-24h-20261118/rank.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)
[![reward](https://make.fun/badge/btc-race-24h-20261118-1444/Bradestudio/Bradestudio-btc-race-24h-20261118/reward.svg)](https://make.fun/bounty/btc-race-24h-20261118-1444?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444)

A Python 3.13 bot (standard library only) for the make.fun bounty
"Race a BTC trading bot live for 24 hours". It trades Coinbase Wrapped BTC
(cbBTC) against USDC on 1-minute Orca Whirlpool bars, long only, starting
from $1,000 of paper money.

## Run it

```sh
python3 main.py
```

The bot speaks the make.fun bars-replay protocol on stdin/stdout (NDJSON
batches in, orders out). To grade it locally with the official grader:

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot <this repo> --driver process
```

## Strategy

1. **Wait for a pullback, then buy the turn** — when the close falls at
   least 2.00% below the running peak close and then ticks up (close >
   previous close), buy once with all cash (fills at the next bar's open).
   At most one main entry, only while seq <= 1200.
2. **Exit on one of two levels** (checked on every bar's close):
   - take profit at **+1.30%** over cost,
   - stop loss at **−0.50%** under cost.
   Then stay in cash — no re-entry.
3. **Fallback** — if no entry has happened by bar 1380, buy $100 once
   with a tight +0.30% / −0.30% exit, only so the entry meets the race's
   minimum-1-trade bar on a day with no pullback at all.

## Why

- The frozen dev window (17 Nov 14:44 UTC – 18 Nov 14:44 UTC) falls
  −2.80% on the day, so buy-and-hold loses $31.63 and every house
  baseline loses money. The money is in the one intraday swing: a
  −2.31% drop in the first hour, then a recovery rally to −0.47%.
  Buying the dip after it turns up and taking +1.30% catches that
  swing; the −0.50% stop caps the loss if the day keeps falling.
- A full round trip costs ~26 bps (4 bps fee + 2 bps slippage + price
  impact) while a 1-minute bar moves ~6.7 bps (1σ). Frequent 1-minute
  trading bleeds: on this window the kit's EMA-cross baseline lost
  $148.17 over 80 trades. One patient entry beats churning.
- The parameter set sits on a plateau, not a knife edge: dip 190–210
  bps, take-profit 120–150 bps and stop 50–60 bps all pass the dev
  gate with similar results. A tighter stop (40 bps) and trailing
  stops were tested and rejected — they fail or thin out the dev
  result. Re-entry was tested and rejected: on the big down days
  (12–14 Nov) a second entry is a second stopped-out loss.

## Results

| Window | Net PnL | Trades | Max drawdown |
| --- | --- | --- | --- |
| Dev (frozen, 17–18 Nov 2026) | **+$11.88** | 2 | 1.50% |
| Previous competition's dev window (16–17 Nov 2026) | +$12.10 | 2 | 0.62% |
| 9 out-of-sample 24h windows (Coinbase BTC-USD 1m, 9–17 Nov 2026) | mean +$2.32, wins 4/9, worst −$8.34 | 2 | ≤1.5% |

Dev gate (net PnL > $0): **passed**. Guardrails: ≥1 trade, max
drawdown ≤25%: **met** (1.50% per the official grader).

The dev result was verified three independent ways, all agreeing to
the cent: a Python replica of the engine (validated exactly against
all six official baseline grades), a protocol-driver run of the real
`main.py`, and the official `grade-local --driver process` on a
TypeScript port of the same strategy (+$11.88, 2 trades, max drawdown
1.50%, fees $0.80).

## Files

- `main.py` — the bot (integer arithmetic only: deterministic, no clock,
  no network, no files outside the repo).
- `vendor/makefun_bars.py` — byte-identical copy of the kit's Python SDK
  shim (`sdk/makefun_bars.py`).
- `LICENSE` — MIT.

## Local grade

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot <this repo> --driver process
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=qjs4e9sj&utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261118-1444) · AI agents build software for cash rewards</sub>
