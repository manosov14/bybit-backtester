# Wave 1 - Risk and Reusable Boundary

## Findings

- Reusable sizing formula: `risk_amount = equity * risk_pct / 100`, `qty = risk_amount / abs(entry - stop)`.
- Entry is `level + entry_ticks * tick` for LONG and `level - entry_ticks * tick` for SHORT.
- Stop has conflicting variants: sweep extreme, fixed ticks from level, or min/max of current and prior spike bars.
- TP is symmetric fixed-RR from entry and stop; operational defaults commonly use `RR=3.0`.
- `domain.risk` is pure, but operational `OrderManager` adds tick/quantity rounding and zero-distance protection.
- Leverage is declared but unused. Fees, slippage, funding, and contract multipliers are absent.
- Minimal reusable boundary excludes exchange auth, DataBroker, scanner state, runtime loops, CLI, execution, and audit logging.
- Bias is not intrinsic to sweep detection. D1 bias only selects/gates ordinary D1 levels in the legacy scanner; H4 bias is not an entry gate.

## Primary sources

- `domain/risk.py`
- `usecases/order_manager.py`
- `usecases/backtest_days.py`
- `domain/bias.py`
- `scanner/filter_adapter.py`
- `scanner/level_watcher.py`
- `strategies/base.py`

All source URLs use pinned commit `152eadffe4abf6948c23b87776cca0d82c539df7`.

## EXPAND

none — formulas and runtime boundaries were fully traced; remaining issues are specification decisions.
