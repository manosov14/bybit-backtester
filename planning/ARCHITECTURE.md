# ARCHITECTURE.md

## Цель
Описать компоненты, Ownership и поток данных MVP Telegram-бота для backtest стратегии false breakout на historical data Bybit. Документ следует `planning/STRATEGY_SPEC.md` и `planning/PRODUCT.md` как единому источнику требований.

## Компоненты и Ownership

### 1. Telegram Bot layer
- **Ownership:** приём команд, парсинг аргументов, отправка отчёта
- **Вход:** Telegram update с командой (`/backtest`, `/strategy`, `/status`, `/help`)
- **Выход:** текстовый отчёт в Telegram или сообщение об ошибке
- **Не делает:** логику strategy, запросы к Bybit/PostgreSQL, расчёт statistics

### 2. Application service
- **Ownership:** оркестрация flow между всеми компонентами
- **Вход:** нормализованные параметры команды из Bot layer
- **Выход:** результат backtest, статус service, описание strategy
- **Делает:** координирует data fetch, strategy execution, engine simulation, persistence и формирование отчёта

### 3. Bybit data layer
- **Ownership:** получение historical candles через public API, нормализация,补全 missing data
- **Вход:** symbol, timeframe, start time, end time
- **Выход:** полный упорядоченный набор closed candles
- **Правила:** сначала PostgreSQL → потом Bybit; только missing ranges; только public API; без auth/WS/real execution

### 4. PostgreSQL repositories
- **Ownership:** хранение candles, backtest runs, trades, results; local cache
- **Вход:** candles из Bybit data layer; run metadata и trades из engine
- **Выход:** candles для заданного range; metadata/status run; trades и results
- **Правила:** candle уникальна по (symbol, timeframe, timestamp); индексы для range queries; repositories скрывают SQL от других слоёв

### 5. Strategy (false breakout)
- **Ownership:** детерминированное генерирование planned signals с entry/SL/TP/risk_per_unit
- **Вход:** D1 candles (closed) для SMA(200) trend и levels, H1 candles/ATR для SpeedRatio, intraday candles (closed), tick_size, entry_ticks, stop mode params, RR, ATR series/data, min_depth_atr, max_depth_atr
- **Выход:** список planned signals с направлением, уровнями, timestamps, depth_abs, depth_atr, planned entry, planned SL, planned TP, risk_per_unit
- **Не делает:**fills, quantity calculation, portfolio state, statistics, exchange interaction
- **Ссылка:** `planning/STRATEGY_SPEC.md` sections 1-9

### 6. Backtest engine
- **Ownership:** симуляция сделок, fills, quantity/price rounding, portfolio state, fees/slippage, statistics
- **Вход:** planned signals от strategy; candles; equity, risk_pct, tick_size, fees/slippage params
- **Выход:** смоделированные trades с entry/exit/result; statistics (number of trades, win rate, Profit Factor, avg R:R, max drawdown, final result)
- **Не делает:**signal generation, entry/SL/TP planning (это strategy)

### 7. Report formatter
- **Ownership:** форматирование statistics и metadata в текстовый отчёт для Telegram
- **Вход:** statistics от engine; run metadata; strategy description
- **Выход:** текстовый отчёт, помещающийся в ограничения Telegram

## Поток данных: `/backtest BTCUSDT 1h 90d`

```
1. Bot layer → Application service: {symbol=BTCUSDT, timeframe=1h, lookback=90d}
2. Application service → Bybit data layer: resolve D1 range + H1 range for SpeedRatio + intraday range
3. Bybit data layer → PostgreSQL: check cached candles
4. PostgreSQL → Bybit data layer: existing candle ranges
5. Bybit data layer → Bybit public API: missing candles only
6. Bybit public API → Bybit data layer: raw candles
7. Bybit data layer → PostgreSQL: persist new candles (normalize timestamps/OHLC)
8. Bybit data layer → Application service: full closed candle sets (D1 + H1 + intraday)
9. Application service → Strategy: D1 trend/levels, H1 SpeedRatio inputs, intraday candles, strategy params
10. Strategy → Application service: planned signals (entry/SL/TP/risk_per_unit per signal)
11. Application service → Engine: signals + candles + equity/risk/fee params
12. Engine → Application service: trades + statistics
13. Application service → PostgreSQL: persist run metadata, trades, results
14. Application service → Report formatter: statistics + metadata
15. Report formatter → Application service: formatted text
16. Application service → Bot layer: report text
17. Bot layer → Telegram: final message
```

## Разделение ответственности: Strategy vs Engine

| Aspect | Strategy | Engine |
|---|---|---|
| Entry/SL/TP planning | Planned levels | Executes at planned levels |
| Risk per unit | Calculates | Uses for quantity |
| Quantity calculation | — | `risk_amount / risk_per_unit` + rounding |
| Position sizing | — | `equity * risk_pct / 100` |
| Fill simulation | — | Checks candles for SL/TP hit |
| Portfolio state | — | Tracks open positions, capital |
| Statistics | — | Calculates all metrics |
| Fees/slippage | — | Applies to trades |

## Multi-timeframe data flow

1. **D1 range resolution:** определяется по lookback period → закрытые D1 candles строят SMA(200) trend state и daily snapshot levels
2. **H1 range resolution:** H1 candles или их ATR series используются для configurable SpeedRatio filter
3. **Intraday range:** определяется по тому же lookback → intraday candles ищут penetration и close-back
4. **Cache check:** PostgreSQL проверяется для всех требуемых timeframes; missing загружается из Bybit
5. **Daily snapshots:** на начало каждого UTC day строится frozen snapshot из закрытых D1 candles (PDL/PDH)
6. **Signal detection:** strategy применяет D1 SMA(200) eligibility gate, SpeedRatio filter и ищет penetration → close-back на intraday candles относительно frozen levels
7. **Engine simulation:** engine берёт planned signals и моделирует trades на тех же candles

## Неразрешённые policy (blocking implementation)

Следующие параметры требуют отдельного решения до реализации:

- timeframe для sweep candles и ATR
- правило преобразования D1 SMA(200) в LONG/SHORT/no-trade
- D1 lookback, количество рабочих PDL/PDH
- ATR length, min/max depth_atr, допустимое окно close-back
- SpeedRatio: delta_price definition, H1 ATR settings, thresholds и enabled/disabled state
- same-bar close-back policy
- stop mode (sweep-extreme / fixed-tick / spike)
- price/tick rounding rules
- position overlap и execution OHLC policy
- fees, slippage, initial capital, risk_pct defaults
- tick_size source для historical run

**Ссылка:** `planning/STRATEGY_SPEC.md` section 10

## Рекомендуемая package structure

```text
src/bybit_backtester/
  bot/              # Telegram Bot layer
  service/          # Application service (orchestration)
  data/             # Bybit data layer
  db/               # PostgreSQL repositories
  strategies/       # Strategy (false breakout)
  engine/           # Backtest engine (simulation)
  report/           # Report formatter
  config.py
  main.py
tests/
```

## Примечания
- Bybit — только read-only data source, без auth и real execution
- Strategy — чистая функция, без side effects и exchange interaction
- Engine владеет simulation, portfolio и statistics; strategy только планирует
- Unresolved policy остаётся explicit blocker до решения владельца
