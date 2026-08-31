# ULW-Research Synthesis: False Breakout Domain Logic

STATUS: complete

## Executive summary

The reference repository does not contain one canonical, working false-breakout strategy. It contains reusable level, sweep-intent, entry, stop, TP, and risk primitives distributed across contradictory legacy paths. The correct migration target is therefore a clean, pure strategy specification rather than a file-for-file port.

The strongest reusable semantics are unbroken D1 PDL/PDH levels, strict penetration plus chronological close-back, ATR-normalized sweep depth, entry offset from the level, explicit stop policy, fixed-RR TP, and linear equity-risk sizing. Bias is not intrinsic to sweep detection and is optional for D1 level selection/gating.

## Findings by theme

- Levels: prior unbroken D1 lows for LONG and highs for SHORT; later inclusive touches invalidate.
- Sweep: strict penetration and strict close-back. The reference pure function has a temporal-order defect and cannot be copied literally.
- Bias: optional D1 level policy only; H4 trend is not required.
- Entry: `level ± entry_ticks * tick_size`.
- Stop: conflicting sweep-extreme, fixed-tick, and spike modes; must remain explicit policy.
- TP: symmetric fixed reward/risk from entry and stop.
- Risk: equity risk divided by absolute entry-stop distance; zero distance invalidates.

## Codebase findings

- Current repo production strategy/engine modules are empty.
- Reference repo pinned at `152eadffe4abf6948c23b87776cca0d82c539df7`.
- Executed fixtures verified levels, sweep edge cases, and risk formulas.

## Sources

- Current repository planning documents and stubs.
- `https://github.com/manosov14/bybit_trade/tree/152eadffe4abf6948c23b87776cca0d82c539df7`.

## Verified claims

- C1-C8 are supported as recorded in `claim-graph.md`.
- Literal correctness of the reference multi-bar sweep implementation is refuted.

## Epistemic instrumentation

- Intent diff closed for domain isolation and bias role.
- Contradictions retained for sweep path, stop mode, rounding, and defaults.
- Runtime proofs recorded as O8-O10.

## Contradictions

- Active scanner has no close-back; pure sweep has close-back but reversed temporal sequencing.
- Stop and rounding policies differ across pure domain, monitor, operational manager, and legacy backtest.

## Gaps

- Numeric defaults, timeframes, same-bar close-back, stop policy, level rearm, and engine OHLC execution remain owner decisions.

## Expansion trace

- Wave 1 mapped source and call chains.
- Wave 2 executed pure functions, counter-searched canonical intent, and reviewed the contract boundary.
- Convergence reached because no discoverable leads remain; remaining forks are explicit policy decisions.
