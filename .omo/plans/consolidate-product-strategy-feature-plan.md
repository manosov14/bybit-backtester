# consolidate-product-strategy-feature-plan - Work Plan

## TL;DR (For humans)

План создаёт единый `planning/PROJECT_SPEC.md` из product scope и false-breakout strategy contract, затем синхронизирует с ним архитектуру, roadmap и feature decomposition. Итоговая документация будет организована вокруг трёх пользовательских outcomes, а не отдельных технических слоёв.

План не меняет production code, не выбирает unresolved trading policies и не включает unrelated `.omo`/`.idea` changes. `PRODUCT.md` и `STRATEGY_SPEC.md` сохраняются как redirect stubs. Старые technical-layer feature files заменяются тремя outcome-based specs только после переноса уникальных требований.

Effort: standard documentation refactor. Главный риск: потеря требований при объединении и появление нескольких sources of truth.

## Scope

### In

- Создание `planning/PROJECT_SPEC.md` как единственного канонического product/strategy source.
- Redirect stubs в `planning/PRODUCT.md` и `planning/STRATEGY_SPEC.md`.
- Актуализация `planning/ARCHITECTURE.md` на основе `PROJECT_SPEC.md`.
- Переработка `planning/ROADMAP.md` как progress tracker по пользовательским outcomes.
- Замена текущих technical-layer feature specs на три outcome-based specs.
- Обновление cross-references в `AGENTS.md`, planning docs и `docs/` index, необходимое по project instructions.
- Documentation QA, cross-reference validation и rendered Markdown review.

### Out

- Любые изменения `src/`, `tests/`, database schema, Docker, runtime или Telegram behavior.
- Выбор unresolved strategy defaults: timeframes, ATR settings, stop mode, RR, same-bar policy, rounding, overlap, fees/slippage, capital/risk defaults.
- Commit или push без отдельного явного запроса пользователя.
- Изменение unrelated `.omo/run-continuation`, `.omo/ulw-research` и `.idea` artifacts.

### Canonical document hierarchy

1. `planning/PROJECT_SPEC.md` - product scope, user flow, strategy contract, formulas, exclusions, success criteria, unresolved policies.
2. `planning/ARCHITECTURE.md` - component ownership and data flow derived from PROJECT_SPEC.
3. `planning/ROADMAP.md` - ordered outcomes, dependencies, progress, cross-cutting work.
4. `planning/features/*.md` - acceptance-level feature specifications.
5. `planning/PRODUCT.md` and `planning/STRATEGY_SPEC.md` - compatibility redirects only.

## Verification strategy

- Test strategy: tests-after for documentation consistency; no code tests because production code is unchanged.
- Every task performs a happy-path and failure-path documentation check and stores evidence under `.omo/evidence/consolidate-product-strategy-feature-plan/`.
- Use `rg` to verify links, stale filenames, required headings, feature numbering, and unresolved-policy preservation.
- Render all changed Markdown to temporary HTML/PDF outside the workspace and inspect every page for malformed lists, clipped Cyrillic, broken links, or orphan headings.
- Reject completion if any authoritative product or strategy rule exists only in a redirect stub or deleted feature file.

## Execution strategy

### Wave 1 - Canonical contract

Tasks 1-3 execute sequentially: inventory requirements, create PROJECT_SPEC, then convert legacy documents to redirects.

### Wave 2 - Derived planning documents

Tasks 4-6 execute sequentially after PROJECT_SPEC: architecture first, then feature decomposition, then roadmap. This keeps one component vocabulary and ensures roadmap links target finalized feature files.

### Wave 3 - References and QA

Tasks 7-8 update references/indexes and run complete documentation verification.

### Dependency matrix

| Task | Depends on | Blocks |
|---|---|---|
| 1 | none | 2 |
| 2 | 1 | 3, 4, 5, 6 |
| 3 | 2 | 7, 8 |
| 4 | 2 | 5, 6, 7, 8 |
| 5 | 2, 4 | 6, 7, 8 |
| 6 | 2, 4, 5 | 7, 8 |
| 7 | 3, 4, 5, 6 | 8 |
| 8 | 1-7 | final verification |

## Todos

- [ ] 1. Inventory every authoritative requirement before consolidation

  **References:** `planning/PRODUCT.md`; `planning/STRATEGY_SPEC.md`; `planning/ARCHITECTURE.md`; `planning/ROADMAP.md`; every file under `planning/features/`; `AGENTS.md`.

  **Work:** Build an evidence table under `.omo/evidence/consolidate-product-strategy-feature-plan/requirements.md` mapping every product command, metric, exclusion, success criterion, strategy rule, formula, invalidation, dependency and unresolved policy to its source line and future destination. Record unrelated dirty paths and exclude them explicitly.

  **Acceptance:** Every heading and non-duplicated requirement from PRODUCT and STRATEGY_SPEC has exactly one planned destination in PROJECT_SPEC. All current feature requirements are marked retain, merge, relocate or obsolete-with-reason.

  **QA happy:** Compare source heading counts and evidence-table coverage; expect no unmapped section.

  **QA failure:** Remove one evidence row in a temporary copy and run the coverage check; expect non-zero exit and the missing source anchor.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/requirements.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 2. Create canonical `planning/PROJECT_SPEC.md`

  **References:** Task 1 evidence; `planning/PRODUCT.md`; `planning/STRATEGY_SPEC.md`.

  **Work:** Merge product summary, user flow, commands, MVP scope, statistics, exclusions and success criteria with strategy inputs, signal rules, LONG/SHORT setup, entry/SL/TP/risk formulas, invalidations, dependency boundary and unresolved policies. Keep Russian prose and established English domain terms. Preserve pinned `bybit_trade` citations. Add a document-status section that names PROJECT_SPEC as canonical and marks unresolved policies as implementation blockers.

  **Acceptance:** PROJECT_SPEC contains all mapped requirements once, not as contradictory duplicates. It explicitly separates confirmed rules, corrected specification choices and unresolved decisions. It contains no production implementation instructions outside planning scope.

  **QA happy:** `rg` verifies required sections and every metric/command/formula; requirement coverage table reports 100% mapped.

  **QA failure:** A duplicate-authority scan detects repeated normative definitions of the same entry/SL/TP rule and fails with both anchors.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/project-spec-check.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 3. Convert PRODUCT and STRATEGY_SPEC into compatibility redirects

  **References:** Approved owner decision in `.omo/drafts/consolidate-product-strategy-feature-plan.md`; completed PROJECT_SPEC.

  **Work:** Replace each legacy file with a short Russian note naming `planning/PROJECT_SPEC.md` as the canonical source, explaining the former document role and linking directly to the relevant PROJECT_SPEC section. Do not retain normative duplicate content.

  **Acceptance:** Both old paths remain valid Markdown files; each contains only redirect context and links. No product or strategy requirement exists exclusively in a stub.

  **QA happy:** Resolve both relative links and confirm target headings exist.

  **QA failure:** Point a temporary stub to a missing heading; link validation must fail and identify the stale anchor.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/redirect-check.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 4. Актуализировать `planning/ARCHITECTURE.md` на основе PROJECT_SPEC

  **References:** `planning/PROJECT_SPEC.md`; current `planning/ARCHITECTURE.md`; requirements evidence.

  **Work:** Define components and ownership for Telegram Bot layer, application service, Bybit data layer, PostgreSQL repositories, strategy, backtest engine and report formatter. Specify the multi-timeframe flow: command parameters -> D1 and intraday range resolution -> cache lookup -> missing Bybit fetch -> normalized closed candles -> frozen daily level snapshots -> pure strategy signals -> engine-owned fills/risk/trades/statistics -> persisted run metadata -> Telegram report. State that strategy owns planned entry/SL/TP and per-unit risk while engine owns execution, quantity, portfolio state and metrics. Preserve unresolved trading policies without defaults.

  **Acceptance:** Every data-flow step has one owning component, explicit input/output and dependency direction. Architecture contains no exchange auth, live runtime, real execution, scanner, unrelated CLI, WebSocket, Redis, ML or multi-strategy design. It links to canonical PROJECT_SPEC sections rather than redirect files.

  **QA happy:** Trace the `/backtest BTCUSDT 1h 90d` scenario end-to-end through all components and verify no responsibility is unowned or duplicated.

  **QA failure:** A boundary check must reject direct Telegram-to-Bybit/SQL calls and strategy-owned fills/statistics.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/architecture-trace.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 5. Replace technical-layer feature list with three user-value feature specs

  **References:** PROJECT_SPEC; updated ARCHITECTURE; current four files under `planning/features/`.

  **Work:** Create `planning/features/01-understand-active-strategy.md`, `02-run-reproducible-backtest.md`, and `03-receive-trusted-result.md`. Feature 1 owns `/strategy`, effective policy visibility and unresolved-policy readiness. Feature 2 owns `/backtest`, multi-timeframe Bybit data, PostgreSQL cache, deterministic signal generation, simulation and reproducibility metadata. Feature 3 owns statistics, no-trade/failure explanations, `/status` and Telegram report interpretation. Each file includes user story, value, scope/non-goals, dependencies, inputs/outputs, rules, edge cases, acceptance criteria and tests. Transfer every unique requirement from the four old technical specs, then remove those replaced files only after coverage passes.

  **Acceptance:** Exactly three numbered outcome feature specs remain. Database, data layer, engine and bot responsibilities are mapped inside features and remain aligned with ARCHITECTURE. Testing and deployment are absent from the feature list and documented as cross-cutting work.

  **QA happy:** Coverage matrix shows every retained old-feature requirement mapped to one new feature or architecture section.

  **QA failure:** Duplicate or orphan requirement detection fails if one responsibility belongs to no feature or conflicting features.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/feature-migration.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 6. Rebuild `planning/ROADMAP.md` as the primary progress tracker

  **References:** PROJECT_SPEC; updated ARCHITECTURE; three new feature specs.

  **Work:** Replace the seven technical stages with an ordered roadmap: foundation/documentation baseline; resolve strategy policy blockers; Feature 1 strategy understanding; Feature 2 reproducible backtest; Feature 3 trusted results; cross-cutting testing; packaging/deployment. For each entry include user value, dependencies, concrete deliverables, exit criteria, links and checkbox status. Keep testing/deployment as steps, not features. Mark only evidence-backed completed work as complete.

  **Acceptance:** Roadmap answers what is next, why it matters, what blocks it and how completion is proven. Feature order matches dependencies and every feature link resolves. Existing implementation stages are not falsely marked completed.

  **QA happy:** Parse roadmap links/statuses and confirm exactly three feature outcomes plus cross-cutting steps.

  **QA failure:** Status validation fails if an implementation outcome is checked complete while its exit criteria or implementation evidence is missing.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/roadmap-check.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 7. Update documentation references and hierarchy guidance

  **References:** `AGENTS.md`; all changed planning files; project rule requiring documentation under `docs/`.

  **Work:** Update AGENTS references from standalone PRODUCT/STRATEGY authority to PROJECT_SPEC and architecture/roadmap hierarchy. Create or update `docs/README.md` as a concise documentation index linking PROJECT_SPEC, ARCHITECTURE, ROADMAP and feature specs, satisfying the existing `docs/` update rule without duplicating normative content. Update stale links in planning files.

  **Acceptance:** Every internal link resolves; no rule instructs agents to update a redirect as canonical content. `docs/README.md` is an index only, not another source of truth.

  **QA happy:** Repository-wide link/reference scan reports only intentional mentions of PRODUCT/STRATEGY redirect files.

  **QA failure:** Introduce a stale canonical reference in a temporary fixture; scan must fail with its path and line.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/reference-check.md`.

  **Commit:** none; commit requires separate user authorization.

- [ ] 8. Run complete documentation consistency and rendered-output QA

  **References:** All deliverables from Tasks 1-7.

  **Work:** Validate required headings, relative links, feature numbering, roadmap statuses, architecture traceability, redirect-only legacy files, unresolved-policy preservation and absence of production-code changes. Render every changed Markdown file to temporary HTML/PDF outside the repository and inspect every page. Record `git diff --name-only` and reject unrelated dirty paths.

  **Acceptance:** All checks pass; no changed production files; no broken links; no clipped Cyrillic or malformed Markdown; no duplicate normative source; evidence artifacts identify exact commands and outputs.

  **QA happy:** Full validation exits zero and rendered pages are readable.

  **QA failure:** Run validators against a temporary broken-link/duplicate-source fixture; both must exit non-zero.

  **Evidence:** `.omo/evidence/consolidate-product-strategy-feature-plan/final-doc-qa.md` plus temporary render manifest.

  **Commit:** none; commit requires separate user authorization.

## Final verification wave

- [ ] F1. Plan compliance audit - verify every approved deliverable and exclusion against the final diff; reject unplanned paths.
- [ ] F2. Documentation quality review - independently check terminology, source hierarchy, contradictions, unresolved policies and requirement coverage.
- [ ] F3. Rendered Markdown QA - inspect every changed document and every page for broken layout, Cyrillic glyphs, links and code blocks.
- [ ] F4. Scope fidelity review - confirm no production code, runtime configuration, unrelated `.omo` artifacts, commit or push was included.

## Commit strategy

No commits are created during execution unless the user separately authorizes git operations. If later requested, use three atomic documentation commits:

1. `docs: consolidate product and strategy specification`
2. `docs: align architecture and feature roadmap`
3. `docs: update documentation references`

## Success criteria

- `planning/PROJECT_SPEC.md` is the sole normative product/strategy source.
- PRODUCT and STRATEGY_SPEC remain valid redirect paths without duplicated requirements.
- ARCHITECTURE fully reflects the canonical multi-timeframe false-breakout flow and component boundaries.
- ROADMAP tracks exactly three user-value features plus cross-cutting testing and deployment steps.
- Feature files are outcome-based, complete, linked and free of orphaned technical responsibilities.
- All unresolved strategy policies remain explicit and block implementation where required.
- Documentation hierarchy and `docs/README.md` index are internally consistent.
- Final verification wave F1-F4 approves with no blockers.
- Production code and unrelated dirty paths remain untouched.
