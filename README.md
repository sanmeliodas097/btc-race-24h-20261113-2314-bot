# Deep-dip BTC bot — make.fun "Race a BTC trading bot live for 24 hours"

Trades CBBTC/USDC on 1-minute bars with $1,000 of paper money.
Python 3.13, standard library only; the kit's Python shim is vendored
byte-for-byte under `vendor/` (excluded from authorship counts).

## Strategy

Long only, one position at a time, full-cash entries, two complementary
modes:

- **DEEP mode (rare, big):** buy when the close is ≥ 3.90% below the
  270-bar (4.5h) high — a multi-hour washout. Exit on +2.50%
  take-profit, a 2.00% trailing stop from the peak since entry, or a
  720-bar (12h) max hold. This is the trade that carries the window:
  it buys the bottom of a -4.9% bleed and exits near the top of the
  +2.9% recovery.
- **REGULAR mode (frequent, small):** buy when the close is ≥ 1.55%
  below the 120-bar (2h) high. Exit on +2.00% take-profit, 0.80%
  trailing stop, or 180-bar hold. A 120-bar cooldown after any exit
  stops the churn that kills dip-reversion in a slow bleed, and a
  deep-bleed guard stands the regular mode down while the price is
  already > 3.00% below the 270-bar high, so it cannot be holding when
  the deep setup fires.

Why: the dev window is a slow low-vol bleed (-4.91% over 24h) with one
V-recovery. Plain dip-reversion loses on it (22 trades, -$61.75) because
it buys every small dip on the way down. Waiting for a genuinely deep
dip, then holding through the recovery, is what pays; the regular mode
exists so the bot still trades (and wins a little) in windows where no
deep dip ever forms. Parameters sit in the middle of a measured plateau
(deep fall 380–400 bps, lookback 240–300 bars, take-profit 220–280 bps,
trailing stop 150–300 bps), not on a knife edge.

## Dev-window results (protocol-level local replay, exact market model)

- **This competition's dev window (1440 bars to 2026-11-13T23:14Z):
  net PnL +$18.05, 6 trades, max drawdown -1.53%** (dev gate: > $0 —
  passes with margin; offline float sim: +$18.08)
- Previous race's dev window (2026-11-12): net PnL +$2.46, 3 trades,
  max drawdown -0.77% — positive on a different regime too.
- Deterministic: identical action logs across 2 runs on each window.

## Files

- `main.py` — the bot (agent-fun/bars-replay/v1, NDJSON stdin/stdout)
- `vendor/makefun_bars.py` — kit Python shim, byte-for-byte copy
- `LICENSE` — MIT · `README.md` — this file
- `.makefun/owner.json` — payout wallet (SOL)
