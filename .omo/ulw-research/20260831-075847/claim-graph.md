# Claim Graph

## Verified claims

Pending research.

| claim_id | statement | type | risk | scope | intent ids | support | contradiction | groups | convergence | counter-search | primary source | dependencies | status | synthesis location |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| C1 | Required strategy logic is separable from live runtime | code | high | domain boundary | I1 | O1, O6 | none | local-contracts, remote-boundary | converged | scanner/runtime counter-trace | source repository | none | supported | STRATEGY_SPEC dependencies |
| C2 | Unbroken prior D1 highs/lows are the reference working levels | code | high | levels | I2 | O2 | unused simpler helper | remote-levels, remote-boundary | converged | searched alternate level builders | source repository | none | supported | STRATEGY_SPEC inputs/rules |
| C3 | Repository contains two incompatible sweep variants, and the pure multi-bar implementation has reversed temporal sequencing | code | high | signals | I2 | O3, O9 | none | remote-levels, verify-sweep | converged | caller and execution counter-search | source repository + runtime proof | C2 | supported | STRATEGY_SPEC rules/ambiguities |
| C4 | Entry is level plus/minus configured ticks | code | high | trade setup | I3 | O5 | none | remote-risk, remote-levels | converged | alternate caller search | source repository | C3 | supported | STRATEGY_SPEC entry |
| C5 | Stop-loss mode differs across sweep extreme, fixed ticks, and spike variants | code | high | trade setup | I3 | O5 | none | remote-risk, remote-boundary | converged | all caller paths inspected | source repository | C3 | supported | STRATEGY_SPEC stop |
| C6 | Take-profit is a fixed RR multiple from entry-to-stop distance | code | high | trade setup | I3 | O4, O5 | none | remote-risk, remote-boundary | converged | alternate formula search | source repository | C4, C5 | supported | STRATEGY_SPEC TP |
| C7 | Risk sizing is equity risk divided by absolute entry-stop distance; zero distance must invalidate sizing | code | high | sizing | I3 | O4, O5, O10 | operational manager substitutes one tick | remote-risk, verify-risk | converged | executed both variants | source repository + runtime proof | C4, C5 | supported | STRATEGY_SPEC risk/ambiguities |
| C8 | Bias is not intrinsic to sweep detection but gates ordinary D1 scanner levels | code | high | filters | I4 | O7, O3 | bias-free setup paths exist | remote-bias, remote-levels | converged | explicit bias-free path search | source repository | C2, C3 | supported | STRATEGY_SPEC dependencies |
