# Wave 2 - Executed Domain Verification

## Level verification

- Executed `working_levels_d1()` at pinned commit `152eadffe4abf6948c23b87776cca0d82c539df7` with pandas 2.3.2 / NumPy 2.3.2.
- Confirmed LONG selects PDL, SHORT selects PDH.
- Later inclusive touches invalidate candidates (`low <= PDL`, `high >= PDH`).
- Last D1 row is excluded as a candidate but participates in invalidation.
- Survivors are returned oldest-first; input itself must already be chronological.
- Invalid direction silently falls back to PDH and must not be adopted.

## Sweep verification

- Executed `check_sweep()` with pandas 2.2.3 / NumPy 2.1.3.
- Confirmed strict penetration and strict close-back inequalities.
- Confirmed ATR depth bounds are inclusive.
- Refuted the function as a correct multi-bar false-break primitive: penetration is checked only on the latest bar while confirmation searches backward, allowing a pre-penetration close to qualify.
- At least `atr_length + 1` bars are required for the reference ATR implementation.
- NaN and zero ATR behavior is unsafe and must become explicit invalidation.

## Risk verification

- Confirmed entry `level ± entry_ticks * tick_size`.
- Confirmed fixed-tick and sweep-extreme stop variants.
- Confirmed symmetric RR TP formulas and `qty = risk_money / abs(entry-stop)`.
- Confirmed leverage is unused.
- Confirmed pure zero-distance behavior returns quantity zero, while operational manager substitutes one tick; the pure behavior is the safe backtest rule.
- Confirmed operational rounding changes effective RR and is execution policy, not core strategy semantics.

## EXPAND

- Closed as unresolved owner decisions: sweep/ATR timeframe, numeric depth defaults, same-bar close-back, stop mode, level rearm, and OHLC execution policy.
