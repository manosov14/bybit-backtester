# Observation Manifest

| observation_id | source | evidence layer | observer group | independence basis | observer | observed_at | valid_at | artifact | anchor | contamination notes |
|---|---|---|---|---|---|---|---|---|---|---|
| O1 | local `src/bybit_backtester/` | workspace | local-contracts | independent local inspection | explore | 2026-08-31 | current workspace | wave-1-explore-current-contracts.md | empty production stubs | none |
| O2 | bybit_trade `domain/levels.py` | source code | remote-levels | independent repository trace | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-levels-sweeps.md | `working_levels_d1` | single-commit repository |
| O3 | bybit_trade `domain/sweep.py`, `scanner/sweep_detector.py` | source code | remote-levels | independent caller comparison | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-levels-sweeps.md | conflicting sweep algorithms | no dedicated tests |
| O4 | bybit_trade `domain/risk.py` | source code + executed examples | remote-risk | independent formula trace | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-risk-boundary.md | sizing and TP formulas | pure examples only |
| O5 | bybit_trade `order_manager.py`, `backtest_days.py` | source code | remote-risk | caller comparison | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-risk-boundary.md | entry/SL/TP variants | legacy paths differ |
| O6 | bybit_trade runtime/scanner/execution modules | source code | remote-boundary | red-team dependency trace | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-risk-boundary.md | exclusion boundary | none |
| O7 | bybit_trade `domain/bias.py`, `scanner/filter_adapter.py` | source code | remote-bias | independent call-chain trace | librarian | 2026-08-31 | commit 152eadf | wave-1-librarian-risk-boundary.md | bias role | no dedicated tests |
| O8 | executed `working_levels_d1` fixtures | runtime proof | verify-levels | isolated pinned execution | deep verifier | 2026-08-31 | commit 152eadf | wave-2-verify-domain-semantics.md | level behavior matrix | temp-only fixtures |
| O9 | executed `check_sweep` fixtures | runtime proof | verify-sweep | isolated pinned execution | deep verifier | 2026-08-31 | commit 152eadf | wave-2-verify-domain-semantics.md | temporal defect and bounds | temp-only fixtures |
| O10 | executed risk/order fixtures | runtime proof | verify-risk | isolated pinned execution | deep verifier | 2026-08-31 | commit 152eadf | wave-2-verify-domain-semantics.md | formula/rounding split | no exchange calls |
| O11 | README/KEEP_FILES/runtime/history counter-search | source + metadata | canonical-counter | independent repository history pass | librarian | 2026-08-31 | commit 152eadf | wave-2-review-contract.md | canonical path unresolved | single root commit |
| O12 | strategy contract review | architecture analysis | contract-review | independent red-team review | oracle | 2026-08-31 | current evidence set | wave-2-review-contract.md | pure signal boundary | policy decisions unresolved |
