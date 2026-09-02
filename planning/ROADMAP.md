# ROADMAP.md

## Назначение

Этот roadmap декомпозирует MVP из `PRODUCT.md` и детализирует его по `mvp_false_breakout_btcusdt.md`.

## Согласованный MVP scope

- Инструмент: только `BTCUSDT` на Bybit Perpetuals.
- Режим: только historical backtest; никаких live orders, private API keys, WebSocket или realtime scanner.
- Стратегия: одна `False Breakout` strategy.
- Команда: `/backtest BTCUSDT <period>`, где `BTCUSDT` фиксирован, а period является единственным изменяемым аргументом после symbol. Пример: `/backtest BTCUSDT 90d`.
- Внутренние timeframes: D1 для trend/levels, H1 для ATR(14)/SpeedRatio, M5 для penetration/close-back и execution.
- Критерий успеха: `/backtest BTCUSDT 90d` возвращает результаты потенциальных setup, причины принятия/отказа, смоделированные trades и итоговую statistics.

## Зафиксированные policy решения

- D1 SMA(200) trend — обязательный eligibility gate для signal.
- D1 state: `close > SMA(200)` даёт LONG, `close < SMA(200)` даёт SHORT, equality даёт no-trade.
- H1 ATR(14), inclusive depth bounds `0.1..0.35`, M5 close-back window в две candles.
- Entry offset 2 ticks, sweep-extreme stop, `RR=3`.
- `risk_pct=1`, не более одной position и SL-first при касании SL/TP в одной execution candle принадлежат engine.
- SpeedRatio остаётся configurable MVP filter на H1.
- Одновременно выполняется один global run. Следующий request получает immediate busy response без queue.
- Application через persistence ports сохраняет candles, runs, candidates с reason codes, trades, results, parameter snapshot и strategy version. PostgreSQL adapter реализует storage, а pure Engine не выполняет I/O.
- Run сохраняет resolved UTC `start`, `end`, `as_of` и immutable association с точным candle set через candle IDs и content/version hash.

## Открытые policy решения

До реализации остаются не определены:

- level lookback N, inside-day policy, количество active levels и rearm;
- SpeedRatio delta origin, threshold и default-enabled state;
- same-bar confirmation и multi-penetration selection;
- execution ordering: первая fill-eligible candle, возможность fill entry на confirmation candle, порядок entry и exit в одной OHLC candle;
- candidate lifecycle: момент создания, multiplicity для одного level, условия termination и precedence между reason codes;
- risk sizing при подтверждённом `risk_pct=1`: initial или current equity как base, интерпретация percentage и sizing formula;
- gap policy, initial capital, fees, slippage и quantity rounding;
- источник `tick_size`, точная ATR warm-up/calculation semantics и price rounding вне entry offset в два ticks;
- metric semantics: Profit Factor при отсутствии losing trades, result при zero trades, basis для maximum drawdown, denominator для average R:R и units для final result.

## Порядок поставки

| # | Feature | Статус | Зависимости |
|---|---|---|---|
| 01 | Bybit historical market data | Ожидает реализации | — |
| 02 | PostgreSQL candle cache | Ожидает реализации | 01 |
| 03 | Market context: D1 trend and working levels | Ожидает определения level policies | 01, 02, policy decisions |
| 04 | False-breakout signals and planned trade | Ожидает определения signal policies | 03, policy decisions |
| 05 | Backtest simulation, explainability and statistics | Заблокировано до определения execution, candidate lifecycle, risk sizing и metric policies | 02, 04, policy decisions |
| 06 | Telegram commands and result delivery | Заблокировано Feature 05 и теми же policy decisions | 05, policy decisions |
| 07 | Testing and reproducibility | Ожидает реализации | 01-06 |
| 08 | Docker delivery | Ожидает реализации | 01-07 |

## Feature 01. Bybit historical market data

**Требование продукта:** получать historical candles через public API Bybit.

Результат: `features/01-bybit-historical-data.md`.

Критерии завершения:
- [ ] Client получает closed BTCUSDT perpetual candles за указанный range.
- [ ] Пагинация, rate limit и ошибки API обработаны явно.
- [ ] Candles нормализованы в UTC и отсортированы от старых к новым.

## Feature 02. PostgreSQL candle cache

**Требования продукта:** кэшировать candles в PostgreSQL; запрашивать только missing data.

Результат: `features/02-postgresql-candle-cache.md`.

Критерии завершения:
- [ ] Candle уникальна по symbol, timeframe и timestamp.
- [ ] Repository возвращает cached range и missing ranges.
- [ ] Повторный запрос уже сохранённого range не вызывает Bybit API.

## Feature 03. Market context: D1 trend and working levels

**Детальные требования MVP:** D1 SMA(200) trend, unbroken extrema N previous D1 candles и configurable internal days.

Результат: `features/03-market-context-and-levels.md`.

Критерии завершения:
- [ ] D1 SMA(200) eligibility gate применён по зафиксированному state rule.
- [ ] По closed D1 candles строятся воспроизводимые PDL/PDH snapshots.
- [ ] Invalidation levels и policy внутреннего дня покрыты tests.

## Feature 04. False-breakout signals and planned trade

**Требования продукта:** одна False Breakout strategy; entry, stop-loss и take-profit для каждого signal.

Результат: `features/04-false-breakout-signals.md`.

Критерии завершения:
- [ ] Penetration -> close-back вычисляется без look-ahead bias.
- [ ] H1 ATR(14), inclusive depth `0.1..0.35`, configurable SpeedRatio и M5 close-back window в две candles применены по зафиксированной policy.
- [ ] Каждый accepted signal содержит direction, level, evidence, entry, SL, TP и risk_per_unit.
- [ ] Entry offset 2 ticks, sweep-extreme stop и `RR=3` рассчитаны детерминированно.
- [ ] Каждый rejected candidate содержит reason code, evidence, parameter snapshot и strategy version context.

## Feature 05. Backtest simulation, explainability and statistics

**Требования продукта:** моделировать trade result по следующим candles; вернуть базовую statistics.

**Статус:** заблокировано до определения execution ordering, candidate lifecycle, risk sizing и metric semantics.

Результат: `features/05-backtest-simulation-and-statistics.md`.

Критерии завершения после снятия policy blockers:
- [ ] Engine моделирует entry, SL, TP и outcome без вызовов Bot/Bybit.
- [ ] Engine применяет `risk_pct=1`, максимум одну position и SL-first при одновременном касании SL/TP.
- [ ] Для каждого candidate доступна evidence trail: accepted/rejected и причина.
- [ ] Runs, candidates, trades и results сохраняются с parameter snapshot и strategy version через Application persistence ports.
- [ ] Run хранит resolved UTC `start`, `end`, `as_of` и immutable association с candle IDs и content/version hash.
- [ ] Рассчитаны восемь метрик из PRODUCT.md.
- [ ] Одинаковые immutable candle set, resolved UTC boundaries, parameters и strategy version дают одинаковый result после correction cache.

## Feature 06. Telegram commands and result delivery

**Требования продукта:** `/backtest`, `/strategy`, `/status`, `/help`; statistics в Telegram.

**Статус:** заблокировано до завершения Feature 05 и определения зависимых policies.

Результат: `features/06-telegram-commands-and-report.md`.

Критерии завершения после завершения Feature 05 и снятия policy blockers:
- [ ] `/backtest BTCUSDT 90d` запускает полный D1/H1/M5 flow и возвращает report.
- [ ] При active run следующий request немедленно получает busy response и не ставится в queue.
- [ ] Async Telegram handler не выполняет CPU simulation напрямую.
- [ ] `/strategy`, `/status` и `/help` отвечают по назначению.
- [ ] Report показывает results, reasons и итоговую statistics.

## Cross-cutting delivery

### Feature 07. Testing and reproducibility

- Unit tests проверяют client normalization, cache ranges, levels, signals, fills и statistics.
- Integration tests проверяют Bybit client, PostgreSQL repositories и полный backtest flow.
- Smoke test проверяет `/backtest BTCUSDT 90d` и immediate busy response для concurrent request.

### Feature 08. Docker delivery

- Dockerfile и docker-compose для app + PostgreSQL.
- Документированный local start.

## Не входит в MVP

Другие symbols/strategies, manual H4 levels, H4 trend, volume delta, live trading, private API keys, realtime scanning, multiple open positions, extended money management, charts, Web UI, ML, Redis и Kubernetes.
