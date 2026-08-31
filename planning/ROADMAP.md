# ROADMAP.md

## Назначение

Этот roadmap декомпозирует MVP из `PRODUCT.md` и детализирует его по `mvp_false_breakout_btcusdt.md`.

## Согласованный MVP scope

- Instrument: только `BTCUSDT` на Bybit Perpetuals.
- Режим: только historical backtest; никаких live orders, private API keys, WebSocket или realtime scanner.
- Strategy: одна `False Breakout` strategy.
- Success: `/backtest BTCUSDT <period>` возвращает результаты потенциальных setup, причины принятия/отказа, смоделированные trades и итоговую statistics.

## Зафиксированные policy решения

- D1 SMA(200) trend — обязательный eligibility gate для signal.
- SpeedRatio — configurable MVP filter на H1; он не переносит realtime scanner или его side effects.

## Открытые policy решения

До Feature 03/04 нужно зафиксировать rule преобразования SMA(200) в trend state, SpeedRatio delta_price/thresholds/enabled state, timeframe для sweep/ATR, D1 lookback, ATR parameters, close-back window, same-bar policy, stop mode, rounding, execution OHLC policy, fees, slippage, capital и risk_pct.

## Порядок поставки

| # | Feature | Статус | Зависимости |
|---|---|---|---|
| 01 | Bybit historical market data | Pending | — |
| 02 | PostgreSQL candle cache | Pending | 01 |
| 03 | Market context: D1 trend and working levels | Pending policy parameters | 01, 02, policy decisions |
| 04 | False-breakout signals and planned trade | Pending policy parameters | 03, policy decisions |
| 05 | Backtest simulation, explainability and statistics | Pending | 02, 04 |
| 06 | Telegram commands and result delivery | Pending | 05 |
| 07 | Testing and reproducibility | Pending | 01-06 |
| 08 | Docker delivery | Pending | 01-07 |

## Feature 01. Bybit historical market data

**Product requirement:** получать historical candles через public API Bybit.

Deliverable: `features/01-bybit-historical-data.md`.

Exit criteria:
- [ ] Client получает closed BTCUSDT perpetual candles за указанный range.
- [ ] Пагинация, rate limit и ошибки API обработаны явно.
- [ ] Candles нормализованы в UTC и отсортированы от старых к новым.

## Feature 02. PostgreSQL candle cache

**Product requirements:** кэшировать candles в PostgreSQL; запрашивать только missing data.

Deliverable: `features/02-postgresql-candle-cache.md`.

Exit criteria:
- [ ] Candle уникальна по symbol, timeframe и timestamp.
- [ ] Repository возвращает cached range и missing ranges.
- [ ] Повторный запрос уже сохранённого range не вызывает Bybit API.

## Feature 03. Market context: D1 trend and working levels

**Detailed MVP requirements:** D1 SMA(200) trend; unbroken extrema N previous D1 candles; configurable internal days.

Deliverable: `features/03-market-context-and-levels.md`.

Exit criteria:
- [ ] D1 SMA(200) eligibility gate применён по зафиксированному state rule.
- [ ] По closed D1 candles строятся воспроизводимые PDL/PDH snapshots.
- [ ] Invalidation levels и policy внутреннего дня покрыты tests.

## Feature 04. False-breakout signals and planned trade

**Product requirements:** одна False Breakout strategy; entry, stop-loss и take-profit для каждого signal.

Deliverable: `features/04-false-breakout-signals.md`.

Exit criteria:
- [ ] Penetration -> close-back вычисляется без look-ahead bias.
- [ ] Depth filter, configurable SpeedRatio filter и close-back window применяются по зафиксированной policy.
- [ ] Каждый accepted signal содержит direction, level, evidence, entry, SL, TP и risk_per_unit.
- [ ] Каждый rejected candidate содержит код причины.

## Feature 05. Backtest simulation, explainability and statistics

**Product requirements:** моделировать trade result по следующим candles; вернуть базовую statistics.

Deliverable: `features/05-backtest-simulation-and-statistics.md`.

Exit criteria:
- [ ] Engine моделирует entry, SL, TP и outcome без вызовов Bot/Bybit.
- [ ] Для каждого candidate доступна evidence trail: accepted/rejected и причина.
- [ ] Рассчитаны восемь метрик из PRODUCT.md.
- [ ] Одинаковые data и parameters дают одинаковый result.

## Feature 06. Telegram commands and result delivery

**Product requirements:** `/backtest`, `/strategy`, `/status`, `/help`; statistics в Telegram.

Deliverable: `features/06-telegram-commands-and-report.md`.

Exit criteria:
- [ ] `/backtest BTCUSDT <period>` запускает полный flow и возвращает report.
- [ ] `/strategy`, `/status` и `/help` отвечают по назначению.
- [ ] Report показывает results, reasons и итоговую statistics.

## Cross-cutting delivery

### Feature 07. Testing and reproducibility

- Unit tests: client normalization, cache ranges, levels, signals, fills, statistics.
- Integration tests: Bybit client, PostgreSQL repositories, full backtest flow.
- Smoke test: `/backtest BTCUSDT <period>`.

### Feature 08. Docker delivery

- Dockerfile и docker-compose для app + PostgreSQL.
- Документированный local start.

## Не входит в MVP

Другие symbols/strategies, manual H4 levels, H4 trend, volume delta, live trading, private API keys, realtime scanning, multiple open positions, extended money management, charts, Web UI, ML, Redis и Kubernetes.
