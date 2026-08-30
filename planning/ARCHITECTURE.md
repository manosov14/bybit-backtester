# ARCHITECTURE.md

## Цель
Сделать систему максимально простой, но при этом способной выполнять real backtest на historical data Bybit.

## Минимальные components
1. **Telegram Bot layer**
   - принимает `/backtest`, `/strategy`, `/status`, `/help`
   - проверяет basic command arguments
   - отправляет итоговый report в Telegram

2. **Backtest engine**
   - получает strategy parameters
   - строит trading signals
   - моделирует trade execution по future candles
   - считает final statistics

3. **Market data layer**
   - сначала читает candles из PostgreSQL
   - запрашивает у Bybit только missing candles
   - сохраняет new candles locally

4. **Database layer**
   - хранит candles и run metadata
   - выступает local cache для уменьшения количества запросов к API

## Поток данных
1. Пользователь отправляет `/backtest BTCUSDT 1h 90d`.
2. Bot разбирает symbol, timeframe и lookback period.
3. Service проверяет PostgreSQL на наличие нужного candle range.
4. Missing candles запрашиваются у public API Bybit и сохраняются.
5. Strategy false breakout генерирует signals.
6. Engine моделирует trades на future candles.
7. Считается statistics.
8. Bot отправляет result в Telegram.

## Рекомендуемая package structure
```text
src/bybit_backtester/
  bot/
  backtest/
  data/
  db/
  strategies/
  config.py
  main.py
tests/
```

## Примечания
- Логику strategy держать отдельно от Telegram и database.
- Рассматривать Bybit только как read-only data source.
- Не усложнять architecture раньше времени: для MVP важнее clarity.
