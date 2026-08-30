# ARCHITECTURE.md

## Goal
Keep the system as small as possible while still performing a real backtest from Bybit historical data.

## Minimal components
1. **Telegram Bot layer**
   - Receives `/backtest`, `/strategy`, `/status`, `/help`
   - Validates basic command arguments
   - Sends the final report back to Telegram

2. **Backtest engine**
   - Resolves strategy parameters
   - Builds trading signals
   - Simulates trade execution on future candles
   - Computes final statistics

3. **Market data layer**
   - Reads candles from PostgreSQL first
   - Fetches only missing candles from Bybit public API
   - Persists new candles locally

4. **Database layer**
   - Stores candles and backtest metadata
   - Acts as the local cache for API minimization

## Data flow
1. User sends `/backtest BTCUSDT 1h 90d`.
2. Bot parses symbol, timeframe, and lookback.
3. Service checks PostgreSQL for the required candle range.
4. Missing candles are fetched from Bybit public API and stored.
5. The false-breakout strategy generates signals.
6. The engine simulates trades using future candles.
7. Statistics are computed.
8. Bot sends the result to Telegram.

## Suggested package layout
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

## Notes
- Keep strategy logic isolated from Telegram and database code.
- Treat Bybit as a read-only data source.
- Avoid premature abstractions; the MVP should optimize for clarity.
