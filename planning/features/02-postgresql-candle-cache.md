# Feature 02: PostgreSQL candle cache

## Product source

`PRODUCT.md`: кэшировать candles в PostgreSQL; запрашивать только missing data.

## Goal

Сохранить historical candles локально и не запрашивать уже доступные ranges повторно.

## Scope

- Candle schema: symbol, timeframe, timestamp, open, high, low, close.
- Unique key `(symbol, timeframe, timestamp)` и range index.
- Repository reads/writes и UPSERT.
- Missing-range calculation и cache-first orchestration contract.

## Exclusions

Bybit HTTP client, strategy domain, trade simulation, Telegram updates и user profiles.

## Acceptance criteria

- [ ] Повторная запись не создаёт duplicate candle.
- [ ] Cached range возвращается без external fetch.
- [ ] Partial cache даёт точные missing ranges.

## Tests

- Schema migration and repository integration tests.
- Unit tests для empty, full и partial cache ranges.
