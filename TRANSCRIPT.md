# Build log — btc-race-24h-20261118-1444 entry

Session: 18 Nov 2026, ~21:55–22:35 WIB. Builder: Muse (Meta AI agent,
working for Andi / @Bradestudio). No submission was made in this session;
the entry was built and graded locally only. A GitHub repository (public,
created by Andi's agent after the competition opened) exists and the
commit is pushed; Andi's confirmation is still needed before any submit.

## Bounty (from make.fun API, 18 Nov 2026 21:52 WIB)

- Slug: btc-race-24h-20261118-1444 — "Race a BTC trading bot live for 24 hours"
- Free (entry fee $0), prize pool $250 (1st $60, 2nd $40, 3rd $30, 4th $25,
  5th $20, 6th–10th $15 each), escrow funded on Solana.
- Deadline: 19 Nov 2026 03:44 WIB (18 Nov 20:44 UTC). Race: 19 Nov ~03:59 WIB –
  20 Nov ~03:59 WIB (24h, 1,440 one-minute bars).
- Bot trades cbBTC/USDC (Orca Whirlpool) with $1,000 paper money, long
  only. Dev gate: net PnL > $0 on the frozen 24h dev window. Race
  guardrails: ≥1 trade, max drawdown ≤25%.
- Same series as btc-race-24h-20261117-0603 (submitted 17 Nov, claim
  AF-QS51Q-2EZF5-3CSQ7-92886), but a different dev window and a different
  strategy: the 0603 strategy (buy early, TP +0.90%, SL −0.60%) would fail
  this window's dev gate, so it was NOT reused.

## Kit

- Downloaded from https://make.fun/c/btc-race-24h-20261118-1444/kit
  (7.52 MB tarball, kit format agent-fun/kit/v1, harness bars-replay/v1).
- Read RULES.txt, the Python SDK shim (sdk/makefun_bars.py, identical to
  the 0603 kit's), the six baseline bots, and the harness source
  (sim/bars/engine.ts) for the exact fill/decision math.
- Dev window: 17 Nov 14:44 UTC – 18 Nov 14:44 UTC, 1,440 bars, 0 gaps.
  Price $65,083 → $63,261 (−2.80%); max intraday drawdown −3.24%.
- Captured the exact wire format with a spy bot through the official
  grade-local (process driver): hello (no warmup_bars), batches seq
  0–1439, done seq 1440. Caps: max 1 open position, 16 orders/bar,
  max weight 10000 bps; fees 4 bps, slippage 2 bps, min notional $1.

## Official baseline grades (kit grade-local, --driver process)

Dev window, 1,440 bars (price $65,083 → $63,261, −2.80%):

| Baseline | Net PnL | Trades | Max DD | Fees |
| --- | --- | --- | --- | --- |
| buy-and-hold | −$31.63 | 1 | 3.25% | $0.79 |
| momentum (house) | −$23.87 | 1 | 3.24% | $0.79 |
| hold-cash | $0.00 | 0 | 0% | $0 |
| ema-cross | −$148.17 | 80 | 14.97% | $29.51 |
| rsi-bollinger | −$39.92 | 19 | 4.35% | $7.89 |
| adx-momentum | −$75.57 | 30 | 7.56% | $11.53 |

Every baseline loses: this is a down window, and the house bots race
but never take a place.

## Simulator validation

A Python replica of the engine (exact integer fill math from engine.ts:
fee 4 bps, slippage 2 bps, constant-product impact vs o_depth, max
slippage vs latest close, decision checks, end flatten) reproduced the
official grades EXACTLY for buy-and-hold (PnL −31,633,698 quote units,
1 trade, fees $0.787502, max DD 3.2484707%) and ema-cross (PnL
−148,174,061, 80 trades, fees $29.510086, max DD 14.9671889%).
Docker is not installed on this machine, so the official grade-local
--driver docker could not be run for the Python bot; the replica + a
protocol-driver test of the real main.py + a TypeScript port graded
with the official grade-local --driver process is the verification
used instead (see below).

## Strategy search

- Family: dip-buy (buy a pullback after it turns up) with take-profit /
  stop-loss, at most 1–2 entries, plus a cheap late fallback trade so
  the entry meets the ≥1-trade bar on a no-pullback day.
- Grid over dip 80–250 bps, reference (day open vs running peak),
  confirmation (none / uptick / bounce off the low), TP 60–150 bps,
  SL 40–80 bps, trailing stop, re-entry 1–3, cooldown, time stop.
- Best dev result: dip 250, TP 120, 2 entries, +$22.73 — but it is
  high-variance out of sample (mean −$7.69 over 9 OOS days, worst
  −$21.64: on the big down days 12–14 Nov a second entry is a second
  stopped-out loss).
- Chosen (R14): dip 200 bps from the running peak close, uptick
  confirmation, TP +1.30%, SL −0.50%, max 1 entry, entry until bar
  1200, no trailing stop, no time stop; fallback at bar 1380 buys $100
  with +0.30% / −0.30% exits. Dev +$11.88, 2 trades, max DD 1.50%.
- Rejected by testing: SL 40 bps (dev −$6.17 — the dev trade's low
  grazes the stop), trailing stops 60/100 bps (dev +$3.17 / −$1.06),
  time stop 300 bars (dev +$1.26), re-entry (worst OOS loss doubles),
  dip 150 bps (OOS mean −$8.00), plain dip-200 without confirmation
  (dev −$7.58: buys mid-fall and stops out).
- Out-of-sample (9 × 24h windows, Coinbase BTC-USD 1m, 9–17 Nov 2026):
  chosen parameters mean +$2.32, wins 4/9, worst −$8.34. The same
  parameters also win +$12.10 on the previous competition's frozen dev
  window (16–17 Nov 2026).
- The fallback never fires on the dev window (the main trade happens
  first); on the two OOS days with no pullback it costs ≈ −$0.53.

## Verification of the submitted commit

Three independent ways, all agreeing to the cent:

1. Python engine replica on the strategy: +$11.884048, 2 trades,
   fees $0.804916, max DD 1.4964%.
2. Protocol-driver run of the real `main.py` (subprocess, exact wire
   messages, run twice): +$11.884048, 2 trades, fees $0.804916,
   max DD 1.4964%, 0 rejected, byte-identical replies both runs
   (deterministic).
3. Official `grade-local --driver process` on a TypeScript port of the
   same strategy: +$11.884048, 2 trades, fees $0.804916, max DD
   1.4963562%, 0 rejected.

Dev gate (net PnL > $0): **PASSED**. Guardrails: ≥1 trade, max
drawdown ≤25%: **met**.

Trades on the dev window: buy decided bar 66, filled bar 67 open
$63,646.15; sell decided bar 406, filled bar 407 open $64,454.10.

## Repository

- https://github.com/Bradestudio/Bradestudio-btc-race-24h-20261118
- Created 18 Nov 2026 14:58 UTC (after the competition opened 14:44 UTC).
- Commit: 8b0a583b7d39ee8d6eb705d5ab6f466dfa0a0b00 (main) at the time of
  writing; the submitted commit is the HEAD of main (only TRANSCRIPT.md
  bookkeeping changed after it).
- About set: website https://make.fun/bounty/btc-race-24h-20261118-1444,
  topics makefun, ai-agents, bounties, trading.
- Files: main.py, vendor/makefun_bars.py (byte-identical kit SDK shim),
  README.md, LICENSE (MIT), TRANSCRIPT.md (this file).

## Still needed before submit (parent / Andi)

- Submit via make.fun (browser): repo URL + commit below + transcript
  bundle.
- **The bundle must be built from the top-level (parent) agent session,
  not from this subagent session.** This session (0ea6dba6-...) is a
  subagent's session: its usage rows are written only once the run
  completes, and the parent session (5d83e8f1-...) is the one that must
  run the bundle (the CLI says so explicitly in its dry-run warning).
  From inside the unpacked kit, the parent runs roughly:
  `node cli/makefun.mjs bundle --repo <entry> --out <bundle.tar.gz>
  --muse-usage <usage-file> --mcp none --harness <name>
  --harness-version <version> --provider meta --model muse-spark-1.3
  --effort default --prompt-file <Andi's words> --prompt-origin scheduled
  --prompt-via "<what started it>" --system-prompt <file>` —
  the usage file comes from a SELECT over the parent's agent_id and
  each subagent's once this run completes. The dry run already finds
  this session (71 turns) and the AGENTS.md instruction file; what it
  still misses and the parent must supply: the person's opening prompt
  (word for word), the system prompt (an entry received from 18 Nov
  2026 12:00 UTC with no system prompt uploaded or carried is
  rejected), the MCP list (none were used in this session), and the
  harness facts (harness, version, provider, model, effort).
- Note: the two 0603 sessions (0a659e31, 2272d17d) must NOT be added:
  they predate this competition's opening and any pre-opening record
  gets the entry rejected (thinking_in_window).
