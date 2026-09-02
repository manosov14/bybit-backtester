# Feature 05: Backtest simulation, explainability and statistics

## Источник продуктовых требований

`PRODUCT.md`: моделировать trade result по следующим candles; вернуть basic statistics.

`mvp_false_breakout_btcusdt.md`: показать каждый potential setup, outcome и reason for rejection.

## Цель

Смоделировать historical trades и выдать проверяемые details и summary.

## Статус

Feature заблокирована до явного определения execution ordering, candidate lifecycle, risk sizing и metric semantics из `STRATEGY_SPEC.md`.

## Scope

- Simulation entry/SL/TP для planned signals.
- Engine владеет `risk_pct=1`, quantity, capital, portfolio state, fees/slippage и execution policy, остаётся pure и не выполняет I/O.
- Не более одной одновременно открытой position.
- При pessimistic execution, если SL и TP затронуты в одной M5 execution candle, SL побеждает.
- Evidence trail для candidate: accepted, rejected, invalidated или trade outcome.
- Application оркестрирует persistence через repository и transaction ports, PostgreSQL adapter реализует storage.
- PostgreSQL хранит `backtest_runs`, `backtest_run_candles`, `signal_candidates`, `trades`, `backtest_results` со strategy version и полным parameter snapshot.
- `backtest_runs` хранит resolved UTC `start`, `end`, `as_of`; `backtest_run_candles` immutable связывает run с candle IDs и content/version hash. Correction создаёт новую candle version, а связанная с run version сохраняется.
- Рассчитываются восемь metrics: trade count, profitable/losing, win rate, Profit Factor, average R:R, maximum drawdown, final result.

## Исключения

Strategy signal generation, Bybit requests, Telegram formatting, real execution и optimization.

## Критерии приёмки

- [ ] Каждая trade хранит entry, SL, TP, exit и outcome.
- [ ] Каждый rejected candidate хранит reason code, evidence и связь с полным parameter snapshot и strategy version.
- [ ] Engine применяет `risk_pct=1`, максимум одну position и SL-first execution policy.
- [ ] Application сохраняет artifacts через repository и transaction ports; Engine не выполняет I/O.
- [ ] Run хранит resolved UTC boundaries и immutable candle-set association; использованные candle versions сохраняются после correction candle cache.
- [ ] Все восемь metrics считаются по документированным rules после определения их semantics.
- [ ] Одинаковые immutable candle set, UTC boundaries, parameters и strategy version дают одинаковые trades и summary.

## Тесты

- Unit tests проверяют fill policy, statistics и zero-trade output после определения policy.
- Unit tests проверяют concurrent signals, same-candle SL/TP и gap policy после её отдельного определения.
- Integration test проверяет flow `signals -> trades -> persisted run` и воспроизведение по immutable candle set.
