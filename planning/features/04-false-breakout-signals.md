# Feature 04: False-breakout signals and planned trade

## Источник продуктовых требований

`PRODUCT.md`: одна False Breakout strategy; entry, stop-loss и take-profit для каждого signal.

`mvp_false_breakout_btcusdt.md`: depth filter, close-back window, entry offset, sweep-extreme stop и RR.

## Зафиксированные решения

- D1 используется для trend/levels, H1 ATR(14) для depth и SpeedRatio, M5 для penetration/close-back.
- Границы depth равны `0.1..0.35` включительно.
- Окно close-back равно двум M5 candles.
- Отступ entry равен 2 ticks, stop использует sweep extreme, `RR=3`.
- `risk_pct=1` и максимум одна position принадлежат engine, а не strategy.

Не разрешены: SpeedRatio delta origin/threshold/default-enabled state, same-bar confirmation, multi-penetration selection, источник `tick_size`, точная ATR warm-up/calculation semantics и price rounding вне двухтикового offset.

## Цель

Преобразовать frozen levels и closed intraday candles в explainable planned signals.

## Scope

- Причинное M5 detection `penetration -> close-back` в окне двух candles.
- H1 ATR(14)-normalized inclusive depth filter `0.1..0.35`.
- Configurable H1 SpeedRatio filter: `SpeedRatio = delta_price / ATR_H1`.
- Planned entry, SL, TP и `risk_per_unit`.
- Evidence accepted/rejected candidate, reason code, parameter snapshot и strategy version context.

## Исключения

Fills, quantity, portfolio state, fees, slippage, live orders и future-data access.

## Критерии приёмки

- [ ] Строгие правила penetration и close-back работают для LONG и SHORT.
- [ ] Invalid candidate содержит конкретный reason code.
- [ ] Accepted signal содержит level, timestamps, depth, entry, SL, TP и RR.
- [ ] Entry offset 2 ticks, sweep-extreme SL и `RR=3` совпадают с contract.
- [ ] Ни один signal не зависит от future candle.

## Тесты

- Unit tests проверяют LONG/SHORT detection, invalidation и calculations.
- Regression tests проверяют look-ahead и same-bar policy.
