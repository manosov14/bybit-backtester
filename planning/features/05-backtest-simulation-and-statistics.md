# Feature 05: Backtest simulation, explainability and statistics

## Product source

`PRODUCT.md`: моделировать trade result по следующим candles; вернуть basic statistics.

`mvp_false_breakout_btcusdt.md`: показать каждый potential setup, outcome и reason for rejection.

## Goal

Смоделировать historical trades и выдать проверяемые details и summary.

## Scope

- Fill/SL/TP simulation from planned signals.
- Engine-owned quantity, capital, portfolio state, fees/slippage и execution policy.
- Candidate evidence trail: accepted, rejected, invalidated или trade outcome.
- Trade/run persistence in PostgreSQL.
- Eight metrics: trade count, profitable/losing, win rate, Profit Factor, average R:R, maximum drawdown, final result.

## Exclusions

Strategy signal generation, Bybit requests, Telegram formatting, real execution и optimization.

## Acceptance criteria

- [ ] Каждая trade хранит entry, SL, TP, exit и outcome.
- [ ] Каждый rejected candidate хранит reason code.
- [ ] Все eight metrics считаются по documented rules.
- [ ] Same inputs and parameters produce identical trades and summary.

## Tests

- Unit tests for fill policy, statistics and zero-trade output.
- Integration test for signals -> trades -> persisted run.
