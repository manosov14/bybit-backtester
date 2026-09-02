# PRODUCT.md

## Описание продукта
MVP Telegram-бота для проверки одной торговой стратегии (trading strategy) на historical data Bybit с выдачей результата прямо в Telegram.

## Основной сценарий
`Telegram command -> historical candles -> trading rules -> trading signals -> trade simulation -> statistics -> response in Telegram`

## Поддерживаемые команды
- `/backtest BTCUSDT <period>`: запустить backtest фиксированного MVP symbol `BTCUSDT` за указанный период, например `/backtest BTCUSDT 90d`
- `/strategy`: показать active strategy, подтверждённые defaults и все unresolved policies
- `/status`: показать service status и доступность data
- `/help`: показать usage help

Аргумент timeframe пользователь не передаёт. В MVP команда принимает только фиксированный symbol `BTCUSDT` и один аргумент периода. Strategy использует внутренние timeframes: D1 для trend и levels, H1 для ATR(14) и SpeedRatio, M5 для penetration и close-back.

## Объём MVP
- получать historical candles через public API Bybit
- кэшировать candles в PostgreSQL
- запрашивать только missing data
- использовать одну strategy: false breakout
- определять для каждого signal entry, stop-loss и take-profit
- моделировать trade result по следующим candles
- применять `risk_pct=1` и ограничение не более одной открытой позиции в backtest engine
- при одновременном касании SL и TP в одной execution candle засчитывать SL
- сохранять candles, параметры запуска, strategy version, signal candidates с reason codes, trades и итоговый result для explainability и воспроизводимости
- сохранять resolved UTC start/end/as-of запуска и immutable связь run с использованным набором candles, чтобы результат воспроизводился после исправления candle cache
- возвращать в Telegram результаты candidates, причины отклонения и базовую statistics

## Выполнение и concurrency
- Во всём приложении одновременно выполняется только один backtest.
- Если active run уже существует, новый запрос сразу получает busy response.
- Дополнительный запрос не ставится в очередь.
- Async Telegram handler передаёт CPU simulation в application execution boundary и не выполняет её напрямую.

## Параметры статистики
- number of trades
- profitable trades
- losing trades
- win rate
- Profit Factor
- average reward-to-risk
- maximum drawdown
- final result

## Что не входит в MVP
- web interface
- charts
- authentication
- WebSocket
- multiple strategies
- machine learning
- Redis
- Kubernetes
- trading API keys
- real order execution

## Критерий успеха
MVP считается готовым, когда пользователь отправляет `/backtest BTCUSDT 90d` в Telegram-бот и получает корректную, объяснимую и воспроизводимую statistics на real historical data Bybit.

## Неразрешённые параметры
До реализации остаются явно не определены:

- level lookback N, inside-day policy, количество одновременно active levels и rearm policy;
- начало `delta_price`, threshold и default-enabled state для SpeedRatio;
- same-bar signal confirmation и выбор при нескольких penetration;
- execution ordering: первая fill-eligible candle, возможность fill entry на confirmation candle, порядок entry и exit в одной OHLC candle;
- candidate lifecycle: момент создания, multiplicity для одного level, условия termination и precedence между reason codes;
- risk sizing при подтверждённом `risk_pct=1`: initial или current equity как base, интерпретация percentage и sizing formula;
- gap policy, initial capital, fees, slippage и quantity rounding;
- источник `tick_size`, точная семантика расчёта и warm-up ATR, а также price rounding вне подтверждённого entry offset в два тика;
- metric semantics: Profit Factor при отсутствии losing trades, result при zero trades, basis для maximum drawdown, denominator для average R:R и units для final result.
