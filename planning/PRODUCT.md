# PRODUCT.md

## Описание продукта
MVP Telegram-бота для проверки одной торговой стратегии (trading strategy) на historical data Bybit с выдачей результата прямо в Telegram.

## Основной сценарий
`Telegram command -> historical candles -> trading rules -> trading signals -> trade simulation -> statistics -> response in Telegram`

## Поддерживаемые команды
- `/backtest` — запустить backtest для symbol, timeframe и периода истории
- `/strategy` — показать active strategy
- `/status` — показать service status и доступность data
- `/help` — показать usage help

## Объём MVP
- получать historical candles через public API Bybit
- кэшировать candles в PostgreSQL
- запрашивать только missing data
- использовать одну strategy: false breakout
- определять для каждого signal entry, stop-loss и take-profit
- моделировать trade result по следующим candles
- возвращать базовую statistics в Telegram

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
MVP считается готовым, когда пользователь отправляет `/backtest BTCUSDT 1h 90d` в Telegram-бот и получает корректную statistics на real historical data Bybit.
