# Спецификация стратегии false breakout

Документ задаёт domain-контракт единственной стратегии MVP до начала реализации. Источником служит `bybit_trade` на зафиксированном commit `152eadffe4abf6948c23b87776cca0d82c539df7` и результаты исполнения его чистых функций. В каталоге `strategies/` на этом commit нет исполняемой стратегии: `base.py` задаёт protocol, а `empty.py` возвращает пустой список signals ([`strategies/base.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/strategies/base.py), [`strategies/empty.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/strategies/empty.py)). Поэтому документ не является буквальным переносом одного strategy-файла.

`domain.sweep.check_sweep` является лучшим семантическим ориентиром для false breakout ([`domain/sweep.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47)), но копировать его буквально нельзя. Исполненная проверка выявила обратный временной порядок: функция ищет close-back в окне, включающем бары до penetration. Ниже причинный порядок исправлен как явный specification choice: penetration должен предшествовать допустимому close-back. Может ли один OHLC bar подтвердить обе фазы, остаётся отдельным решением из раздела 10. Активный scanner фиксирует только пересечение уровня и не требует close-back, поэтому он не является семантическим источником false breakout ([`scanner/sweep_detector.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/scanner/sweep_detector.py#L21-L97)).

## 1. Обязательные входные данные

Стратегия получает:

- `symbol` и направление `LONG` или `SHORT`;
- D1 candles в хронологическом порядке, от старых к новым, с полями `timestamp`, `open`, `high`, `low`, `close`; используются только закрытые candles;
- закрытые intraday candles для поиска penetration и close-back с теми же OHLC-полями, также в хронологическом порядке;
- закрытые H1 candles или эквивалентную ATR series для SpeedRatio filter;
- D1 SMA(200) trend state, детерминированно рассчитанный только по закрытым D1 candles;
- конечный `tick_size > 0`, целый `entry_ticks >= 0`, параметры выбранного stop mode и конечный `RR > 0`;
- ATR series либо данные и параметры её детерминированного расчёта, а также конечные границы `0 <= min_depth_atr <= max_depth_atr`;
- `equity` и `risk_pct` не являются входами strategy; их получает backtest engine для расчёта размера позиции.

Набор OHLC-полей следует контрактам reference-функций ([`domain/levels.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L19-L30), [`domain/sweep.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L31)). Хронологический порядок является обязательным specification choice: reference-код полагается на позиционный порядок, но не проверяет его. Требование использовать только закрытые бары также является исправленным specification choice: reference-код не проверяет, что intraday candle уже закрыта, а level builder позиционно считает последнюю D1 строку открытой. `tick_size` передаётся как готовый domain-вход. Стратегия не загружает exchange metadata и не обращается к CCXT.

## 2. Правила генерации signal

1. В начале каждого UTC day `D` строится и фиксируется daily snapshot уровней из D1 candles, период которых завершился не позднее начала `D`. Candle `D-1`, закрывшаяся на границе начала `D`, включается в snapshot. Для `LONG` кандидат равен PDL, то есть `low` закрытого D1 bar. Для `SHORT` кандидат равен PDH, то есть `high` закрытого D1 bar.
2. Более позднее закрытое D1 касание, уже доступное при построении snapshot, включительно отменяет кандидат: для PDL достаточно `later.low <= PDL`, для PDH достаточно `later.high >= PDH`.
3. Открытая D1 candle не участвует ни как кандидат, ни как источник invalidation. Если input содержит только закрытые D1 bars, последняя строка является допустимым кандидатом. Это осознанное уточнение reference-алгоритма: он всегда исключает последнюю строку, потому что неявно считает её текущей открытой candle ([`domain/levels.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L52-L71)). Snapshot не меняется intraday и тем самым не удаляет уровень задним числом после sweep.
4. Неверное направление не получает fallback. Допустимы только `LONG` и `SHORT`; reference-поведение, где любое значение кроме `LONG` выбирает PDH, не принимается.
5. Для каждого рабочего уровня false breakout определяется причинной последовательностью `penetration -> close-back`. Close-back ищется начиная со следующего закрытого bar либо с penetration bar, если отдельно разрешена same-bar policy. Проверка идёт слева направо и видит только закрытые бары не позднее текущего шага. Future data запрещены.
6. `LONG penetration`: `low < level`. `SHORT penetration`: `high > level`. Равенство не считается penetration.
7. `LONG close-back`: `close > level`. `SHORT close-back`: `close < level`. Равенство не подтверждает возврат.
8. `depth_abs = level - penetration_low` для `LONG`, `depth_abs = penetration_high - level` для `SHORT`; `depth_atr = depth_abs / ATR`. Обе заданные границы ATR включительны. ATR должен быть конечным и строго положительным.
9. D1 SMA(200) trend является обязательным eligibility gate: LONG candidates рассматриваются только при состоянии D1 `LONG`, SHORT candidates — только при состоянии D1 `SHORT`. H4 trend в MVP не используется. Правило преобразования SMA(200) в `LONG`/`SHORT`/no-trade state должно быть отдельно зафиксировано до реализации.
10. SpeedRatio является configurable filter MVP. Он рассчитывается на H1: `SpeedRatio = delta_price / ATR_H1`. Кандидат должен содержать рассчитанное значение и результат фильтра; rule выбора начала `delta_price`, ATR parameters, threshold и enabled/disabled state остаются policy parameters.

Семантика PDL, PDH и inclusive invalidation подтверждена [`working_levels_d1`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L32-L72). Closed-only snapshot устраняет неявную позиционную модель открытой последней строки в reference-коде. Строгие inequalities и inclusive ATR bounds взяты из [`check_sweep`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47). Причинный порядок является исправленным specification choice по доказанному temporal defect этой функции. Необязательность bias подтверждается тем, что D1 gating находится во внешнем scanner filter, а не в sweep primitive ([`scanner/filter_adapter.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/scanner/filter_adapter.py#L24-L56)).

## 3. LONG setup

LONG setup существует, когда выполнены все условия:

1. Есть действующий PDL, построенный по правилам раздела 2.
2. Закрытый intraday bar строго проникает ниже PDL: `penetration_low < level`.
3. Глубина penetration попадает в обе включительные ATR-границы.
4. На допустимом по same-bar policy закрытом bar происходит строгий close-back: `close > level`. Допустимость same-bar подтверждения остаётся открытым решением из раздела 10.
5. До момента подтверждения не возникло ни одного условия invalidation из раздела 8.

Signal фиксирует как минимум `symbol`, `direction=LONG`, identity и цену PDL, время penetration, время confirmation, sweep extreme, `depth_abs`, `depth_atr`, planned entry, planned SL, planned TP, `risk_per_unit` и planned `RR`. Направление и условия penetration/close-back опираются на [`working_levels_d1`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L52-L72) и [`check_sweep`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47); временной порядок исправлен спецификацией.

## 4. SHORT setup

SHORT setup существует, когда выполнены все условия:

1. Есть действующий PDH, построенный по правилам раздела 2.
2. Закрытый intraday bar строго проникает выше PDH: `penetration_high > level`.
3. Глубина penetration попадает в обе включительные ATR-границы.
4. На допустимом по same-bar policy закрытом bar происходит строгий close-back: `close < level`. Допустимость same-bar подтверждения остаётся открытым решением из раздела 10.
5. До момента подтверждения не возникло ни одного условия invalidation из раздела 8.

Signal фиксирует тот же набор полей, что LONG setup, с `direction=SHORT` и PDH. Направление и условия penetration/close-back опираются на [`working_levels_d1`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L52-L72) и [`check_sweep`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47); временной порядок исправлен спецификацией.

## 5. Расчёт entry

Planned entry задаётся относительно уровня:

```text
entry_LONG  = level + entry_ticks * tick_size
entry_SHORT = level - entry_ticks * tick_size
risk_per_unit = abs(entry - stop)
risk_amount = equity * risk_pct / 100
quantity_raw = risk_amount / risk_per_unit
```

Первые две формулы подтверждены reference order plan ([`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L414-L455)). Формулы риска и количества подтверждены чистым domain-кодом ([`domain/risk.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26)). `risk_per_unit == 0` отменяет setup и запрещает деление. Подстановка одного tick вместо нулевой дистанции из operational path не принимается.

Strategy может рассчитать planned entry и `risk_per_unit`, но не исполняет вход. `risk_amount` и `quantity_raw` приведены как обязательные формулы общего контракта; их применение, quantity rounding, проверка доступного capital и portfolio state принадлежат backtest engine.

## 6. Расчёт stop-loss

Source содержит три несовместимых stop variants. До выбора stop mode реализация стратегии не считается полностью определённой:

- **Sweep-extreme mode:** `stop_LONG = sweep_low`, `stop_SHORT = sweep_high`.
- **Fixed-tick mode:** `stop_LONG = level - stop_ticks * tick_size`, `stop_SHORT = level + stop_ticks * tick_size`.
- **Spike mode:** `stop_LONG = min(penetration_bar.low, previous_bar.low)`, `stop_SHORT = max(penetration_bar.high, previous_bar.high)`.

Sweep-extreme и fixed-tick варианты реализованы в [`OrderManager.make_plan`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L414-L455). Spike variant встречается в legacy backtest ([`usecases/backtest_days.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/backtest_days.py#L89-L107)). Спецификация сохраняет конфликт и не назначает default. После выбора mode обязательна направленная проверка: `stop < entry` для `LONG`, `stop > entry` для `SHORT`.

## 7. Расчёт take-profit

Take-profit симметрично отстоит от entry на выбранный reward-to-risk multiple:

```text
TP_LONG  = entry + RR * (entry - stop)
TP_SHORT = entry - RR * (stop - entry)
```

Формулы предполагают прошедшую направленную проверку stop из раздела 6 и совпадают с чистой реализацией [`position_sizing`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26) и operational plan ([`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L414-L455)). Числовое значение `RR` не задаётся этим документом. Strategy рассчитывает planned TP, а engine решает, был ли он фактически достигнут.

## 8. Условия invalidation

Кандидат уровня, незавершённый setup или готовый signal отклоняется, если применимо хотя бы одно условие:

- при построении следующего daily snapshot D1 candidate получил более позднее закрытое D1 inclusive touch: `low <= PDL` либо `high >= PDH`; intraday snapshot остаётся frozen, чтобы сам sweep не удалял уровень задним числом;
- входные candles идут не в хронологическом порядке, содержат незакрытый bar в core detection или требуют future data для подтверждения;
- penetration отсутствует, касается уровня только равенством или не получает допустимый close-back в выбранном временном окне;
- ATR отсутствует, не является конечным положительным числом либо `depth_atr` выходит за включительные границы;
- направление не равно `LONG` или `SHORT`;
- stop находится с неверной стороны entry;
- `risk_per_unit = abs(entry - stop)` равен нулю.

Inclusive invalidation уровня следует [`_later_bars_breaks`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L32-L44). Строгие penetration и close-back, а также ATR bounds следуют [`check_sweep`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47). Явная отмена при невалидном ATR и неверном направлении является corrected specification choice: `check_sweep` использует небезопасный ATR fallback ([`domain/sweep.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L35-L40)), а `working_levels_d1` трактует любое направление, кроме `LONG`, как `SHORT` ([`domain/levels.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L52-L58)). Отмена при open/future bars является отдельным требованием причинности. Нулевая дистанция отменяет setup вместо разных source-поведений в [`position_sizing`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26) и [`OrderManager.make_plan`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L451-L455).

## 9. Зависимости от существующей domain-логики bybit_trade

Нужно переиспользовать на уровне семантики, с новой чистой реализацией:

- выбор unbroken D1 PDL/PDH и inclusive invalidation из [`domain/levels.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/levels.py#L32-L72);
- строгие penetration, close-back и ATR-normalized depth из [`domain/sweep.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/sweep.py#L22-L47), но только с исправленным причинным порядком;
- entry, sweep-extreme и fixed-tick stop variants, symmetric RR TP и linear risk formulas из [`domain/risk.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26) и [`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L414-L455); spike variant отдельно происходит из [`usecases/backtest_days.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/backtest_days.py#L89-L107);
- границу pure strategy output без exchange side effects, отражённую в [`SignalIntent`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/strategies/base.py#L25-L42).

Strategy отвечает за детерминированное signal generation и может вернуть planned entry, SL, TP, `risk_per_unit`, timestamps и доказательства setup. Backtest engine отвечает за fills, price и quantity rounding, portfolio state, capital/risk application, position overlap, trade outcomes, fees, slippage и statistics. Это сохраняет отделение strategy от симуляции, уже заложенное в текущей архитектуре MVP.

Не входят в перенос и не являются зависимостями стратегии: real order execution, authentication, live runtime, scanner-only functionality, unrelated CLI, exchange metadata, CCXT, scanner state, sessions, delta filter, order placement и audit logging. D1 SMA(200) trend gate и configurable H1 SpeedRatio входят в MVP как explicit policy, но не переносят scanner pipeline или его side effects. В приведённых диапазонах `OrderManager` смешивает exchange initialization, metadata, rounding и planning ([`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L370-L462)), а scanner pipeline смешивает depth, session и speed filters ([`scanner/filter_adapter.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/scanner/filter_adapter.py#L59-L150)). Эти components не нужны чистому backtest domain.

## 10. Неразрешённые неоднозначности

До реализации владелец стратегии должен отдельно выбрать и зафиксировать следующие policy. Этот документ намеренно не подставляет source defaults:

- timeframe для sweep candles и timeframe для ATR;
- правило преобразования D1 SMA(200) в states `LONG`, `SHORT` или no-trade;
- D1 lookback и правило выбора количества одновременно рабочих PDL/PDH;
- ATR length, минимальную и максимальную `depth_atr`, допустимое окно close-back и все числовые defaults;
- правило определения `delta_price`, H1 ATR parameters, thresholds и enabled/disabled state для SpeedRatio filter;
- допустим ли same-bar close-back; OHLC одного bar не доказывает, что penetration случился раньше close;
- если до close-back есть несколько penetration bars, какой из них задаёт penetration timestamp, sweep extreme, `depth_abs` и используемое значение ATR;
- один stop mode из sweep-extreme, fixed-tick и spike variants;
- правила price/tick rounding;
- может ли уровень rearm после invalidation или уже обработанного sweep;
- position overlap и execution OHLC policy, включая порядок SL/TP при касании обоих уровней в одном bar и обработку gaps;
- fees, slippage, initial capital и `risk_pct` defaults;
- источник `tick_size` для исторического запуска и quantity rounding policy engine.

Эти развилки существуют из-за несовместимых retained paths: pure risk не округляет значения ([`domain/risk.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/domain/risk.py#L18-L26)), operational path округляет price и quantity ([`usecases/order_manager.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/order_manager.py#L399-L455)), а legacy backtest задаёт собственный stop и последовательность SL/TP ([`usecases/backtest_days.py`](https://github.com/manosov14/bybit_trade/blob/152eadffe4abf6948c23b87776cca0d82c539df7/usecases/backtest_days.py#L89-L126)). Ни один из этих вариантов не объявляется default без отдельного решения.
