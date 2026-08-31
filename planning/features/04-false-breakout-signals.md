# Feature 04: False-breakout signals and planned trade

## Product source

`PRODUCT.md`: одна False Breakout strategy; entry, stop-loss и take-profit для каждого signal.

`mvp_false_breakout_btcusdt.md`: depth filter, close-back window, entry offset, sweep-extreme stop и RR.

## Decision gate

До реализации нужно зафиксировать intraday/ATR timeframe, ATR parameters, close-back window, same-bar policy, multi-penetration policy, SpeedRatio delta_price/thresholds/enabled state, stop mode, tick rounding и RR.

## Goal

Преобразовать frozen levels и closed intraday candles в explainable planned signals.

## Scope

- Causal penetration -> close-back detection.
- ATR-normalized depth filter.
- Configurable H1 SpeedRatio filter: `SpeedRatio = delta_price / ATR_H1`.
- Planned entry, SL, TP, `risk_per_unit`.
- Accepted/rejected candidate evidence и reason code.

## Exclusions

Fills, quantity, portfolio state, fees, slippage, live orders и future-data access.

## Acceptance criteria

- [ ] Strict penetration and close-back rules work for LONG and SHORT.
- [ ] Invalid candidate exposes a specific reason code.
- [ ] Accepted signal contains level, timestamps, depth, entry, SL, TP and RR.
- [ ] No signal depends on a future candle.

## Tests

- Unit tests for LONG/SHORT detection, invalidation and calculations.
- Regression tests for look-ahead and same-bar policy.
