# Feature 06: Telegram commands and result delivery

## Источник продуктовых требований

`PRODUCT.md`: `/backtest`, `/strategy`, `/status`, `/help`; response in Telegram.

## Цель

Дать пользователю Telegram interface для запуска и понимания historical BTCUSDT backtest.

## Статус

Feature заблокирована до завершения Feature 05 и определения execution, candidate lifecycle, risk sizing и metric policies, от которых зависит корректный report.

## Scope

- `/backtest BTCUSDT <period>` принимает фиксированный `BTCUSDT` и только period как изменяемый аргумент. Пример: `/backtest BTCUSDT 90d`.
- Flow использует внутренние D1, H1 и M5 timeframes. Пользователь не передаёт timeframe.
- Во всём приложении active только один run. Параллельный request сразу получает busy response и не ставится в queue.
- `/strategy` показывает active strategy и resolved/unresolved policies.
- `/status` показывает availability of service/data.
- `/help` показывает usage and examples.
- Report содержит candidate results, rejection reasons и final statistics.
- Async handler передаёт CPU simulation application layer и не запускает её напрямую в event loop.

## Исключения

Console interface, `/run`, live scanner, `/levels`, `/trend`, `/param`, authentication и trading controls.

## Критерии приёмки

- [ ] `/backtest BTCUSDT 90d` возвращает читаемый report.
- [ ] Команда с timeframe argument отклоняется с корректной usage guidance.
- [ ] Параллельный request получает immediate busy response без queue.
- [ ] Report содержит все statistics из `PRODUCT.md` и explainability data из Feature 05.
- [ ] Invalid command arguments возвращают usage guidance.
- [ ] Bot handlers не обращаются напрямую к Bybit, SQL или strategy logic.
- [ ] Bot handlers не выполняют CPU simulation напрямую.

## Тесты

- Unit tests проверяют parsing команд и форматирование output.
- Full-flow integration test использует mocked application service.
- Concurrency test проверяет active run и immediate busy response.
