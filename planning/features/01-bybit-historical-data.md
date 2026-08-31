# Feature 01: Bybit historical market data

## Product source

`PRODUCT.md`: получать historical candles через public API Bybit.

## Goal

Получить historical closed candles для `BTCUSDT` Bybit Perpetuals без private API keys.

## Scope

- Bybit public market-data client.
- Запрос klines за symbol, timeframe и time range.
- Pagination, нормализация OHLC/timestamp, chronological ordering.
- Явная обработка empty response, rate limit и API failure.

## Exclusions

Private API, live data, WebSocket, order placement, PostgreSQL persistence, strategy и Telegram.

## Acceptance criteria

- [ ] Client возвращает только closed candles в UTC, от старых к новым.
- [ ] Большой period корректно разбивается на API pages.
- [ ] Ошибка или отсутствие данных не маскируется под успешный backtest.

## Tests

- Unit tests для normalization, ordering и pagination.
- Integration test against Bybit public API.
