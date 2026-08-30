# Bybit data layer

## Назначение
Получать реальные historical candles через public API Bybit и дополнять локальный cache только отсутствующими данными.

## Объём
- читать доступный candle range из PostgreSQL
- определять missing ranges для запроса
- получать candles через public API Bybit
- приводить ответ API к внутреннему формату
- сохранять новые candles и возвращать полный запрошенный range

## Что не входит
- private API и trading API keys
- WebSocket и live market data
- real order execution
- расчёт strategy и statistics

## Входные данные
- symbol, timeframe, start time и end time
- candles, уже сохранённые в PostgreSQL
- ограничения public API Bybit на размер и частоту запросов

## Выходные данные
- полный упорядоченный набор candles для запрошенного range
- новые candles, сохранённые в local cache
- понятная ошибка, если market data получить нельзя

## Правила
- сначала проверять PostgreSQL, затем обращаться к Bybit
- запрашивать только missing ranges
- учитывать pagination и rate limits public API
- нормализовать timestamps и числовые поля до сохранения
- повторная загрузка не создаёт дубликаты candles
- возвращать candles в хронологическом порядке

## Пограничные случаи
- запрошенный range полностью есть в cache
- отсутствуют данные в начале, середине или конце range
- API возвращает пустой ответ
- ответ API содержит пересекающиеся страницы или неполную последнюю страницу
- Bybit временно недоступен или сообщает о rate limit

## Критерии приёмки
- полный cached range возвращается без запроса к Bybit
- missing candles загружаются и становятся доступны для следующего запроса
- для одной комбинации symbol, timeframe и timestamp хранится одна candle
- ошибки внешнего API не маскируются под пустой успешный результат

## Тесты
- unit tests для расчёта missing ranges и нормализации ответа
- integration tests для Bybit client и database repository
- сценарии полного, частичного и пустого cache
- проверка pagination, rate limit и пустого ответа
- повторный запрос не создаёт дубликаты
