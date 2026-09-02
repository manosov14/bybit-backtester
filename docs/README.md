# Документация MVP

## Authoritative hierarchy

1. [PRODUCT.md](../planning/PRODUCT.md): пользовательский contract, scope и критерий успеха. При изменении user-facing behavior обновляется первым.
2. [STRATEGY_SPEC.md](../planning/STRATEGY_SPEC.md): domain-контракт strategy и полный список resolved и unresolved parameters.
3. [ARCHITECTURE.md](../planning/ARCHITECTURE.md): components, dependency direction, execution flow и persistence contract.
4. Раздел `# MVP` в [mvp_false_breakout_btcusdt.md](../planning/mvp_false_breakout_btcusdt.md): подробный MVP scope. Разделы до `# MVP` являются historical vision и не authoritative для текущего MVP.
5. [ROADMAP.md](../planning/ROADMAP.md) и [feature specifications](../planning/features/): delivery sequence и acceptance criteria, которые должны соответствовать документам выше.

При конфликте применяется документ, который владеет соответствующим contract: `PRODUCT.md` для user-facing behavior и scope, `STRATEGY_SPEC.md` для strategy rules, `ARCHITECTURE.md` для system boundaries и execution. Roadmap и feature specs не переопределяют эти contracts.

## Правило сопровождения

Каждое изменение кода, конфигурации или документации обязано одновременно обновлять всю затронутую техническую документацию в `docs/` и `planning/`.

- User-facing change: сначала обновить `planning/PRODUCT.md`, затем остальные affected documents и implementation.
- Architecture change: обновить `planning/ARCHITECTURE.md` и все остальные affected documents.
- Strategy change: обновить `planning/STRATEGY_SPEC.md` и согласовать detailed MVP, roadmap и affected feature specs.
- Unspecified parameters должны оставаться явно unresolved. Нельзя добавлять не подтверждённые defaults.
