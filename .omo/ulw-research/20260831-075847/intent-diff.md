# Intent vs Reality

| intent_id | expected truth | observed reality | diff | violated invariant | intent source | supporting observations | status | claim ids |
|---|---|---|---|---|---|---|---|---|
| I1 | False-breakout strategy can be isolated from live trading runtime | Pure domain modules exist, while scanner/runtime/execution are separable orchestration | none | none | user request | O1, O6 | true | C1 |
| I2 | Levels and sweep detection have traceable rules in `bybit_trade` | Level rules are traceable; two contradictory sweep variants exist | sweep variant must be selected explicitly | deterministic strategy | user request | O2, O3 | violated | C2, C3 |
| I3 | Entry, stop-loss, take-profit, and risk rules can be specified without exchange execution | Pure formulas exist, but stop mode and rounding differ by caller | caller variants require explicit MVP choice | deterministic calculations | user request | O4, O5 | violated | C4, C5, C6, C7 |
| I4 | Bias or trend is included only when proven necessary by the source call chain | Bias is not intrinsic to sweep; it only selects/gates D1 levels in legacy scanner | include only if reproducing scanner level policy | minimal dependency | user request | O7 | true | C8 |
