# Verification Economics

| claim | risk | error cost | verification cost | chosen path | decision | outcome | residual risk |
|---|---|---|---|---|---|---|---|
| Strategy domain dependency map | high | Incorrect backtest semantics | medium | Cross-trace definitions, callers, and tests | verify | pure boundary confirmed; canonical runtime unresolved | explicit policy choices remain |
| Entry / SL / TP formulas | high | Incorrect simulated PnL | medium | Read source formulas and execute focused examples | verify | formulas confirmed; stop/rounding variants differ | numeric defaults unresolved |
| Bias requirement | medium | Unnecessary scope or missing filter | low | Trace actual call chain and counter-search unused paths | verify | not intrinsic; optional D1 policy | user may later enable policy |
| Sweep temporal semantics | high | Look-ahead or false signals | medium | Execute causal-order fixtures | verify | literal source behavior refuted | spec must correct ordering |
