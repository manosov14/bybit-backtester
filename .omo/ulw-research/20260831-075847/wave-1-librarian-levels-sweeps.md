# Wave 1 - Levels and Sweeps

## Findings

- Reference repository pinned at `152eadffe4abf6948c23b87776cca0d82c539df7`.
- `domain.levels.working_levels_d1()` selects unbroken prior daily lows for LONG and highs for SHORT, excluding candidates broken by later bars.
- Active `scanner.SweepDetector` classifies the latest M1 bar by its deeper penetration side and does not require close-back.
- Pure `domain.sweep.check_sweep()` requires strict penetration, ATR depth bounds, and a close back across the level within `max_closeback_bars`.
- `check_sweep()` is the explicit reusable false-breakout primitive; the live scanner path is orchestration/state-heavy and behaviorally different.
- No dedicated tests resolve which sweep variant is canonical.

## Primary sources

- `domain/levels.py`
- `domain/levels_engine.py`
- `domain/sweep.py`
- `scanner/sweep_detector.py`
- `scanner/runner.py`
- `scanner/filter_adapter.py`

All source URLs use pinned commit `152eadffe4abf6948c23b87776cca0d82c539df7`.

## EXPAND

- LEAD: direction and candle-close semantics — WHY: code mixes vocabularies and evaluates the last bar without proving it is closed — ANGLE: define canonical direction and closed-bar contract.
