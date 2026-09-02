# ARCHITECTURE.md

## Цель

MVP реализуется как modular monolith с ports and adapters. Он запускает одну false breakout strategy для `BTCUSDT`, получает real historical data из public Bybit API, сохраняет воспроизводимый результат в PostgreSQL и возвращает отчёт через Telegram.

## Модули

```text
src/bybit_backtester/
  domain/              # candles, levels, candidates, signals, trades, results
  strategies/          # чистая False Breakout strategy
  engine/              # fills, positions, risk, execution, statistics
  application/         # use cases, orchestration, single-run guard
  ports/               # контракты market data, repositories, clock, execution
  adapters/
    bybit/              # public REST historical market data
    postgres/           # PostgreSQL repositories
  bot/                  # Telegram handlers, parser, report formatter
  bootstrap/            # composition root и config loading
```

## Направление зависимостей

Зависимости направлены внутрь:

```text
bot, adapters, bootstrap -> application -> strategies, engine, ports -> domain
```

- `domain` не зависит от других модулей и внешних frameworks.
- `strategies` и `engine` зависят только от `domain`.
- `application` координирует domain services через `ports`.
- `adapters/bybit`, `adapters/postgres` и `bot` реализуют внешние детали и не вызываются из domain напрямую.
- `bootstrap` связывает implementations с ports и загружает config.

## Ownership

### Domain

Содержит immutable model и правила целостности для candles, levels, candidates, signals, trades и results. Не выполняет I/O.

### Strategies

Строит D1 context и levels, применяет H1 filters, обнаруживает M5 false breakout и выдаёт planned signal или rejected candidate с evidence и reason code. Strategy не рассчитывает quantity и не управляет position state.

### Engine

Применяет `risk_pct=1`, разрешает не более одной открытой позиции, моделирует fills и outcomes, считает statistics. Если SL и TP затронуты в одной M5 execution candle, SL имеет приоритет. Engine является pure и не выполняет I/O или persistence orchestration.

### Application

Оркестрирует data loading, strategy, engine, persistence и report preparation. Application владеет глобальным single-run guard и persistence transactions через repository и transaction ports.

### Ports

Определяют интерфейсы public market data, candle cache, run persistence, transaction boundary и вынесенного CPU execution.

### Adapters

`adapters/bybit` получает closed candles через public Bybit REST API без trading keys. `adapters/postgres` реализует repository и transaction ports, хранит candles и backtest artifacts, но не владеет orchestration.

### Bot

Парсит `/backtest`, `/strategy`, `/status`, `/help` и форматирует ответы. Handler не содержит strategy, SQL или Bybit logic.

### Bootstrap и config

Создаёт application graph, подключает adapters и читает подтверждённые и ещё не разрешённые параметры из config после их явного определения.

## Command contract и timeframe flow

Пользователь запускает:

```text
/backtest BTCUSDT 90d
```

`BTCUSDT` является фиксированным symbol MVP. Единственный изменяемый аргумент команды после symbol: период. Timeframe не передаётся пользователем.

Внутренний flow:

```text
D1 closed candles -> SMA(200) trend state + frozen PDL/PDH levels
H1 closed candles -> ATR(14) depth normalization + SpeedRatio
M5 closed candles -> penetration + close-back within two candles
planned signals -> M5 engine execution -> trades + statistics
```

Подтверждённые strategy defaults: D1 `close > SMA(200)` означает LONG, `close < SMA(200)` означает SHORT, equality означает no-trade; depth bounds `0.1..0.35` включительны; entry offset равен 2 ticks; stop ставится за sweep extreme; `RR=3`.

## Backtest flow

1. Bot передаёт application request `{symbol=BTCUSDT, period=90d}`.
2. Application атомарно проверяет global active run.
3. Если run уже active, application немедленно возвращает busy response. Запрос не ставится в очередь.
4. Application разрешает точные UTC `start`, `end`, `as_of` и определяет D1, H1 и M5 ranges с нужным warm-up.
5. PostgreSQL adapter возвращает cached candles и missing ranges.
6. Bybit adapter загружает только missing closed candles и сохраняет их через repository port.
7. Strategy формирует accepted и rejected candidates с evidence, reason codes, parameter snapshot и strategy version context.
8. Engine моделирует не более одной позиции, применяет `risk_pct=1` и pessimistic SL-first policy.
9. Application через repository и transaction ports сохраняет run, immutable candle-set association, candidates, trades и result в одной согласованной transaction flow.
10. Bot получает подготовленный report и отправляет его в Telegram.
11. Application освобождает active-run guard после success или failure.

## Concurrency и async boundary

Во всей системе одновременно active только один backtest. Application атомарно обеспечивает global guard. Дополнительный request сразу получает busy response и не ожидает lock или queue.

Telegram handler остаётся async, но не выполняет CPU simulation напрямую в event loop. Он вызывает application port, который переносит CPU-bound engine execution в worker thread или process boundary. Конкретный механизм до реализации может быть выбран без изменения domain contract.

## Persistence model

PostgreSQL содержит следующие logical tables:

| Table | Назначение |
|---|---|
| `candles` | Versioned closed D1, H1 и M5 OHLC data; correction создаёт новую immutable version вместо изменения version, связанной с run |
| `backtest_runs` | Command period, resolved UTC `start`, `end`, `as_of`, status, timestamps, полный parameter snapshot и strategy version |
| `backtest_run_candles` | Immutable association run с точными candle IDs и content/version hash использованного candle set |
| `signal_candidates` | Accepted и rejected candidates, evidence, reason code, полный parameter snapshot и strategy version напрямую или через immutable связь с run |
| `trades` | Entry, SL, TP, quantity, exit, outcome и execution evidence |
| `backtest_results` | Итоговые metrics и связь с completed run |

Полный parameter snapshot, strategy version и resolved UTC boundaries сохраняются на уровне run. `backtest_run_candles` не меняется после фиксации run и ссылается на immutable candle versions. Correction создаёт новую candle version, а использованная run version сохраняется, поэтому можно восстановить точный input после исправления cache. Candidate хранит собственные evidence и reason code, чтобы rejection и acceptance можно было воспроизвести. Application управляет persistence flow через ports, PostgreSQL adapter только реализует storage. Schema details, data types и indexes определяются при проектировании adapter, не меняя logical contract.

## Deployment

Docker Compose содержит только два services:

- `app`: Telegram bot и modular monolith;
- `postgres`: PostgreSQL для candle cache и backtest persistence.

Redis, Web UI, WebSocket, Kubernetes и отдельные microservices не входят в MVP.

## Неразрешённые параметры

До реализации остаются явно не определены:

- level lookback N;
- inside-day policy;
- количество одновременно active levels;
- rearm policy;
- начало `delta_price`, threshold и default-enabled state SpeedRatio;
- same-bar signal confirmation;
- выбор при нескольких penetration;
- execution ordering: первая fill-eligible candle, возможность fill entry на confirmation candle, порядок entry и exit в одной OHLC candle;
- candidate lifecycle: момент создания, multiplicity для одного level, условия termination и precedence между reason codes;
- risk sizing при подтверждённом `risk_pct=1`: initial или current equity как base, интерпретация percentage и sizing formula;
- gap policy;
- initial capital;
- fees;
- slippage;
- quantity rounding;
- источник `tick_size`;
- точная семантика ATR warm-up и расчёта;
- любое price rounding, не заданное entry offset в два ticks;
- metric semantics: Profit Factor при отсутствии losing trades, result при zero trades, basis для maximum drawdown, denominator для average R:R и units для final result.

Ни один из этих параметров не получает неявного default.
