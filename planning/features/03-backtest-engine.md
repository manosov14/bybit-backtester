# Backtest engine

## Назначение
Проверять strategy false breakout на historical candles и возвращать воспроизводимый результат симуляции сделок.

## Объём
- принимать параметры backtest и упорядоченный набор candles
- находить trading signals по правилам strategy
- определять entry, stop-loss и take-profit
- моделировать результат каждой сделки по следующим candles
- рассчитывать итоговую statistics

## Что не входит
- поддержка нескольких strategies
- оптимизация параметров strategy
- real order execution
- загрузка market data и отправка ответа в Telegram

## Входные данные
- symbol, timeframe и границы периода
- historical candles с timestamp, open, high, low, close и volume
- однозначно заданные параметры strategy

## Выходные данные
- список смоделированных trades
- number of trades, profitable trades и losing trades
- win rate, Profit Factor и average reward-to-risk
- maximum drawdown и final result

## Правила
- candles обрабатываются в хронологическом порядке
- signal строится только по данным, доступным на момент его появления
- одна и та же candle не должна повторно открывать одну и ту же сделку
- правила entry, stop-loss, take-profit и закрытия сделки фиксируются до реализации
- одинаковые входные данные и параметры дают одинаковый результат

## Пограничные случаи
- данных недостаточно для расчёта signal
- в периоде нет signals или закрытых trades
- stop-loss и take-profit достигнуты внутри одной candle
- последняя сделка не закрыта к концу периода
- в последовательности candles есть пропуск

## Критерии приёмки
- engine запускает backtest без обращения к Telegram, Bybit API или database
- каждая trade содержит данные входа, выхода и результат
- statistics рассчитывается по завершённым trades по зафиксированным формулам
- неоднозначные сценарии исполнения описаны и покрыты явными правилами

## Тесты
- unit tests для правил strategy и расчёта каждого показателя
- сценарии с прибыльной, убыточной и незакрытой trade
- сценарий без signals
- проверка отсутствия look-ahead bias
- повторный запуск на одинаковых данных возвращает одинаковый результат
