# Feature 03: Market context and working levels

## Product source

`mvp_false_breakout_btcusdt.md`: D1 SMA(200) trend и рабочие экстремумы N previous days без снятой liquidity.

## Зафиксированное решение

D1 SMA(200) — обязательный eligibility gate MVP: LONG candidates допустимы только в D1 `LONG` state, SHORT candidates — только в D1 `SHORT` state. H4 trend не входит в MVP. Отдельно требуется зафиксировать правило преобразования SMA(200) в state `LONG`/`SHORT`/no-trade.

## Goal

По closed D1 BTCUSDT candles детерминированно определить market context и frozen working PDL/PDH levels.

## Scope

- D1 SMA(200) trend state как обязательный gate.
- Configurable D1 lookback N.
- PDL/PDH candidates из closed D1 candles.
- Inclusive invalidation: `low <= PDL`, `high >= PDH`.
- Configurable inclusion of inside days.
- Frozen daily snapshots для intraday detection.

## Exclusions

H4 trend, manual H4 levels, realtime recalculation и strategy execution.

## Acceptance criteria

- [ ] Snapshot использует только data, closed до начала UTC day.
- [ ] Каждый working level сохраняет identity, side и invalidation evidence.
- [ ] D1 SMA(200) state rule и inside-day policy зафиксированы до implementation.

## Tests

- Unit tests для SMA trend, level selection, inside days и invalidation.
- Determinism test: same D1 input gives same snapshot.
