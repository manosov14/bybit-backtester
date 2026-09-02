# Спецификация стратегии false breakout

Документ задаёт domain-контракт единственной стратегии MVP. Strategy работает только с `BTCUSDT` и закрытыми candles. Она детерминированно формирует accepted signals и rejected candidates, но не управляет capital, positions или исполнением сделок.

## 1. Внутренние timeframes

- D1: trend state и working PDL/PDH levels.
- H1: ATR(14) для depth filter и SpeedRatio.
- M5: penetration, close-back и execution candles.

Пользователь не выбирает timeframe в команде. Период запуска задаётся через `/backtest BTCUSDT <period>`, например `/backtest BTCUSDT 90d`.

## 2. D1 trend state

Trend state рассчитывается только по последней закрытой D1 candle и SMA(200), рассчитанной только по закрытым D1 candles:

- если `close > SMA(200)`, state равен `LONG`;
- если `close < SMA(200)`, state равен `SHORT`;
- если `close == SMA(200)`, действует no-trade state.

LONG candidates рассматриваются только в `LONG` state. SHORT candidates рассматриваются только в `SHORT` state. H4 trend в MVP не используется.

## 3. Working levels

В начале каждого UTC day строится frozen snapshot рабочих D1 levels из данных, закрытых не позднее начала этого дня.

- Для LONG используется PDL, то есть `low` закрытой D1 candle.
- Для SHORT используется PDH, то есть `high` закрытой D1 candle.
- Более позднее доступное D1 касание включительно отменяет level: `low <= PDL` или `high >= PDH`.
- Открытая D1 candle не участвует в выборе или invalidation.
- Snapshot не меняется intraday и не удаляет level задним числом после sweep.

Level lookback N, inside-day policy, количество одновременно active levels и rearm policy пока не определены.

## 4. M5 false breakout

False breakout задаётся причинной последовательностью `penetration -> close-back` на закрытых M5 candles без future data.

- LONG penetration: `low < level`.
- SHORT penetration: `high > level`.
- LONG close-back: `close > level`.
- SHORT close-back: `close < level`.
- Равенство не считается penetration или close-back.
- Close-back должен произойти в окне двух M5 candles.

Допустимость confirmation на той же M5 candle, выбор при нескольких penetration и точная трактовка границ двухсвечного окна относительно same-bar policy пока не определены.

## 5. H1 ATR и depth filter

Для нормализации глубины используется ATR(14) на H1:

```text
depth_abs_LONG = level - penetration_low
depth_abs_SHORT = penetration_high - level
depth_atr = depth_abs / ATR_H1(14)
```

Допустимый диапазон `depth_atr` равен `0.1 <= depth_atr <= 0.35`. Обе границы включительны. ATR должен быть конечным и строго положительным.

Точная семантика ATR warm-up и расчёта пока не определена.

## 6. SpeedRatio

SpeedRatio является configurable filter MVP и рассчитывается на H1:

```text
SpeedRatio = delta_price / ATR_H1(14)
```

Каждый candidate сохраняет рассчитанное значение и результат фильтра. Начало `delta_price`, threshold и default-enabled state пока не определены.

## 7. Planned entry, stop-loss и take-profit

Entry задаётся на расстоянии двух ticks от рабочего level:

```text
entry_LONG = level + 2 * tick_size
entry_SHORT = level - 2 * tick_size
```

Stop-loss всегда использует фактический sweep extreme:

```text
stop_LONG = sweep_low
stop_SHORT = sweep_high
```

Take-profit использует фиксированный `RR=3`:

```text
TP_LONG = entry + 3 * (entry - stop)
TP_SHORT = entry - 3 * (stop - entry)
```

Strategy отклоняет candidate, если stop находится не с той стороны entry или `risk_per_unit = abs(entry - stop)` равен нулю. Источник `tick_size` и любое price rounding, не заданное двухтиковым offset, пока не определены.

## 8. Граница Strategy и Backtest Engine

Strategy отвечает за levels, filters, signal evidence и planned entry, SL, TP, `risk_per_unit`. Backtest engine отвечает за:

- `risk_pct=1` и расчёт quantity;
- не более одной одновременно открытой позиции;
- fills, portfolio state и trade outcomes;
- fees, slippage и rounding;
- pessimistic execution: если SL и TP затронуты в одной M5 execution candle, результатом считается SL;
- statistics.

Engine является pure domain component и не выполняет I/O. Initial capital, equity base, percentage interpretation, sizing formula, fees, slippage, quantity rounding, execution ordering, metric semantics и gap policy пока не определены.

## 9. Invalidation и reason codes

Candidate отклоняется с конкретным reason code, если применимо хотя бы одно условие:

- отсутствует допустимый trend state или направление ему не соответствует;
- level недействителен;
- нет strict penetration;
- нет close-back в подтверждённом окне;
- ATR отсутствует, не положителен, не конечен или depth находится вне включительного диапазона;
- SpeedRatio filter отклоняет candidate;
- stop находится не с той стороны entry;
- `risk_per_unit` равен нулю;
- для решения потребовались незакрытые или будущие candles.

Набор reason codes должен быть стабильным и пригодным для хранения, отчёта и regression tests.

## 10. Persistence и reproducibility

Persistence orchestration принадлежит Application, которое работает через repository и transaction ports. PostgreSQL adapter реализует storage. Strategy и Engine остаются pure и не выполняют I/O. Для каждого run сохраняются:

- полный parameter snapshot, включая engine и strategy parameters;
- strategy version;
- resolved UTC `start`, `end` и `as_of`;
- immutable association с точным candle set через candle IDs и content/version hash; использованная candle version сохраняется, а correction создаёт новую version;
- accepted и rejected signal candidates;
- reason code, evidence, полный parameter snapshot и strategy version для каждого rejected candidate, напрямую или через immutable связь с run snapshot;
- planned signals, trades и итоговый backtest result.

Одинаковые immutable candle set, resolved UTC boundaries, parameter snapshot и strategy version должны давать одинаковый результат даже после исправления candle cache.

## 11. Application concurrency

Ограничение одного active backtest относится к application layer. Если run уже выполняется, новый запрос получает немедленный busy response и не ставится в очередь.

## 12. Неразрешённые параметры

Следующие параметры остаются явно не определены и не получают неявных defaults:

- level lookback N;
- inside-day policy;
- количество одновременно active levels;
- rearm policy;
- начало `delta_price`, threshold и default-enabled state для SpeedRatio;
- same-bar signal confirmation;
- выбор candidate при нескольких penetration;
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
- любое price rounding, не заданное двухтиковым entry offset;
- metric semantics: Profit Factor при отсутствии losing trades, result при zero trades, basis для maximum drawdown, denominator для average R:R и units для final result.

В MVP не входят H4 trend или levels, другие symbols или strategies, live trading, private API keys, WebSocket и real order execution.

## 13. Provenance и reference implementation

Domain-контракт основан на `bybit_trade` commit `152eadffe4abf6948c23b87776cca0d82c539df7`, но не копирует его буквально. Причинный порядок `penetration -> close-back`, closed-only candles и явная обработка invalid states являются исправленными specification choices.

- PDL/PDH и inclusive invalidation: [`domain/levels.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L32-L72).
- Penetration, close-back и ATR-normalized depth: [`domain/sweep.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47).
- Risk и position sizing formulas: [`domain/risk.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26).
- Entry, sweep-extreme/fixed-tick stop variants и operational rounding: [`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L399-L455).
- Legacy spike stop и SL/TP sequence: [`usecases/backtest_days.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/backtest_days.py#L89-L126).
- Pure strategy boundary: [`strategies/base.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/strategies/base.py#L25-L42).

Reference code не является source of truth для подтверждённых MVP defaults. При расхождении действуют правила этого документа и `PRODUCT.md`.
Формулы risk sizing и execution sequence из reference code являются только provenance: пока соответствующие policy перечислены как unresolved, они не выбирают MVP semantics.
