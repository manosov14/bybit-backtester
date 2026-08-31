# Wave 1 - Current Repository Contracts

## Findings

- Production modules are empty stubs; all strategy requirements are planning-only.
- The normalized candle contract is `timestamp`, `open`, `high`, `low`, `close`, `volume`, plus `symbol`, `timeframe`, and requested range.
- The strategy must be pure and deterministic, without Telegram, Bybit API, PostgreSQL, or metric calculation dependencies.
- Existing planning requires no look-ahead but leaves signal shape, timing, SL/TP precedence, position overlap, gaps, warm-up, and metric formulas unresolved.

## Sources

- `planning/features/03-backtest-engine.md`
- `planning/features/02-bybit-data-layer.md`
- `planning/features/01-database-model.md`
- `planning/PRODUCT.md`
- `planning/ARCHITECTURE.md`
- empty modules under `src/bybit_backtester/`

## EXPAND

- LEAD: strategy-to-engine contract — WHY: signals and trades lack defined fields and lifecycle — ANGLE: specify typed inputs/outputs and no-look-ahead timing.
- LEAD: deterministic OHLC execution — WHY: same-candle SL/TP and gaps otherwise make results irreproducible — ANGLE: define conservative execution rules.
