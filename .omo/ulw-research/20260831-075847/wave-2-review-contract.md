# Wave 2 - Contract and Canonical-Path Review

## Findings

- Canonical executable strategy is not provable: repository intent points to a new `strategies/` implementation, but the only strategy is empty.
- `domain.sweep.check_sweep()` is the best semantic reference, not a safe implementation to copy.
- Strategy should consume closed bars as a chronological stream and emit immutable setup signals.
- Portfolio state, fills, quantity rounding, fees, overlap, OHLC ambiguity, and trade outcomes belong to the backtest engine.
- Bias is not needed for core false-breakout detection; it remains an optional D1 level-selection/gating policy.
- Required signal evidence includes level identity, direction, penetration/confirmation times, sweep extreme, ATR/depth, entry trigger, stop, TP, and planned RR.

## Counter-search result

README, KEEP_FILES, runtime, scanner, monitor, backtest, branches, tags, issues, and commit history were checked. The remaining uncertainty is intrinsic to contradictory retained legacy code and a single root commit.

## EXPAND

none — remaining items are explicitly unresolved product/trading-policy decisions, not discoverable repository facts.
