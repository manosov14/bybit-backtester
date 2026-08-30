# Database model

## Назначение
Хранить historical candles и результаты backtest в PostgreSQL, а также служить local cache для market data.

## Объём
- определить таблицы candles, backtest runs, trades и results
- зафиксировать поля, типы данных, ключи и связи
- обеспечить уникальность candles
- добавить индексы для чтения candle ranges и результатов run
- определить границы database repositories

## Что не входит
- Redis и другие cache systems
- хранение Telegram updates и пользовательских профилей
- market data вне historical candles
- аналитическое хранилище и долгосрочная агрегация

## Входные данные
- нормализованные candles из Bybit data layer
- параметры и статус backtest run
- смоделированные trades и итоговая statistics

## Выходные данные
- candles для заданных symbol, timeframe и range
- metadata и статус backtest run
- связанные trades и result
- границы доступного candle range для поиска missing data

## Правила
- candle уникальна по symbol, timeframe и timestamp
- timestamps хранятся в едином формате и timezone
- числовые торговые значения сохраняются без потери требуемой точности
- trades и result связаны с конкретным backtest run
- запись результата run выполняется согласованно с записью его trades
- repositories скрывают SQL от bot layer и backtest engine

## Пограничные случаи
- повторное сохранение той же candle
- частично пересекающиеся candle ranges
- backtest run завершился ошибкой до появления result
- run завершён без trades
- удаление или изменение связанных записей

## Критерии приёмки
- схема содержит минимальные сущности candles, backtest runs, trades и results
- constraints не допускают дубликаты candles и потерю связей
- индексы поддерживают основные запросы без полного просмотра таблиц
- repository может сохранить и прочитать полный результат одного run
- failed run сохраняет статус без фиктивного result

## Тесты
- migration test для создания схемы с нуля
- integration tests для записи и чтения каждой сущности
- проверка unique constraint для candles
- проверка связей между run, trades и result
- проверка запросов полного и частичного candle range
