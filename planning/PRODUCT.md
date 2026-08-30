# PRODUCT.md

## Product summary
An MVP Telegram bot that backtests a single trading strategy on historical Bybit data and returns results directly in chat.

## Primary user flow
`Telegram command -> historical candles -> strategy rules -> trading signals -> trade simulation -> statistics -> Telegram response`

## Supported commands
- `/backtest` — run a backtest for a symbol, timeframe, and lookback window
- `/strategy` — show the active strategy
- `/status` — show service and data availability status
- `/help` — show usage help

## MVP scope
- Source historical candles from Bybit public API
- Cache candles in PostgreSQL
- Request only missing data
- Run one strategy: false breakout
- Derive entry, stop-loss, and take-profit for each signal
- Simulate trade outcomes from subsequent candles
- Return core statistics in Telegram

## Statistics to return
- number of trades
- profitable trades
- losing trades
- win rate
- Profit Factor
- average reward-to-risk
- maximum drawdown
- final result

## Explicitly out of scope
- Web interface
- charts
- authentication
- WebSocket
- multiple strategies
- machine learning
- Redis
- Kubernetes
- trading API keys
- real order execution

## Success criteria
The MVP is complete when a user can send `/backtest BTCUSDT 1h 90d` to the Telegram bot and receive correct statistics based on real Bybit historical data.
