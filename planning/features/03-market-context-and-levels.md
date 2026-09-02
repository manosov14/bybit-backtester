# Feature 03: Market context and working levels

## Источник продуктовых требований

`mvp_false_breakout_btcusdt.md` задаёт D1 SMA(200) trend и рабочие экстремумы N previous days без снятой liquidity.

## Зафиксированное решение

D1 SMA(200) является обязательным eligibility gate MVP. Для последней закрытой D1 candle `close > SMA(200)` даёт `LONG`, `close < SMA(200)` даёт `SHORT`, equality даёт no-trade. H4 trend не входит в MVP.

## Цель

По closed D1 BTCUSDT candles детерминированно определить market context и frozen working PDL/PDH levels.

## Scope

- D1 SMA(200) trend state как обязательный gate.
- Настраиваемый D1 lookback N.
- PDL/PDH candidates из closed D1 candles.
- Включительная invalidation: `low <= PDL`, `high >= PDH`.
- Настраиваемое участие inside days.
- Frozen daily snapshots для intraday detection.

## Исключения

H4 trend, manual H4 levels, realtime recalculation и strategy execution.

## Критерии приёмки

- [ ] Snapshot использует только candles, закрытые до начала UTC day.
- [ ] Каждый working level сохраняет identity, side и invalidation evidence.
- [ ] D1 SMA(200) state rule реализован без look-ahead.
- [ ] Неопределённые level lookback N, inside-day policy, количество active levels и rearm policy зафиксированы до implementation соответствующей логики.

## Тесты

- Unit tests проверяют SMA trend, level selection, inside days и invalidation.
- Тест на детерминизм подтверждает, что одинаковые D1 input дают одинаковый snapshot.
