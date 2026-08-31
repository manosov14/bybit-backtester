# Feature 06: Telegram commands and result delivery

## Product source

`PRODUCT.md`: `/backtest`, `/strategy`, `/status`, `/help`; response in Telegram.

## Goal

Дать пользователю Telegram interface для запуска и понимания historical BTCUSDT backtest.

## Scope

- `/backtest BTCUSDT <period>` запускает data -> strategy -> simulation flow.
- `/strategy` показывает active strategy и resolved/unresolved policies.
- `/status` показывает availability of service/data.
- `/help` показывает usage and examples.
- Report includes candidate results, rejection reasons and final statistics.

## Exclusions

Console interface, `/run`, live scanner, `/levels`, `/trend`, `/param`, authentication и trading controls.

## Acceptance criteria

- [ ] `/backtest BTCUSDT <period>` returns a readable report.
- [ ] Report includes all PRODUCT.md statistics and explainability data from Feature 05.
- [ ] Invalid command arguments return usage guidance.
- [ ] Bot handlers do not access Bybit, SQL or strategy logic directly.

## Tests

- Unit tests for command parsing and output formatting.
- Full-flow integration test with a mocked application service.
