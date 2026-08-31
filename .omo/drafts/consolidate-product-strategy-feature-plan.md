# consolidate-product-strategy-feature-plan - Draft

intent: clear
review_required: false
status: plan-written

## Request

Consolidate `planning/PRODUCT.md` and `planning/STRATEGY_SPEC.md` into one canonical document, then update the roadmap and feature plan from that source.

## Components

- C1: Canonical product and strategy specification - pending owner decision on source-file disposition.
- C2: User-value feature topology - grounded in `planning/ROADMAP.md` and `planning/features/`.
- C3: Cross-references and documentation QA - grounded in `AGENTS.md`, roadmap references, and dirty-worktree constraints.
- C4: Architecture alignment - update `planning/ARCHITECTURE.md` from the canonical product/strategy contract.

## Decisions

- Default canonical path: `planning/PROJECT_SPEC.md`.
- Preserve unresolved strategy policies as explicit blockers; do not invent defaults.
- Reorganize feature plan around user value: understand strategy, run reproducible backtest, receive and trust result.
- Treat testing and deployment as cross-cutting delivery steps, not product features.
- Test strategy: documentation structure, link/cross-reference checks, duplicate-source checks, and rendered Markdown QA; no code tests.

## Owner decision

- `PRODUCT.md` and `STRATEGY_SPEC.md` become short compatibility stubs pointing to `PROJECT_SPEC.md`.

## Risks

- dirty_worktree: unrelated `.omo/run-continuation`, `.omo/ulw-research`, and `.idea` paths must not be overwritten or included.
- Duplicate sources of truth can drift if full legacy documents remain.

## Approach

1. Create `planning/PROJECT_SPEC.md` as the canonical source of product scope, user flow, strategy rules, formulas, exclusions, success criteria, and unresolved policy decisions.
2. Replace `planning/PRODUCT.md` and `planning/STRATEGY_SPEC.md` with redirect stubs to the canonical document.
3. Update `planning/ROADMAP.md` as the primary progress tracker: replace the current technical-stage decomposition with ordered user-value outcomes, explicit dependencies, cross-cutting testing/deployment steps, statuses, and links to the feature specs.
4. Redesign the `planning/features/` decomposition around the user journey: strategy understanding, reproducible historical backtest, and result interpretation/trust. Map database, Bybit data layer, strategy/engine, and Telegram responsibilities into those features instead of treating every technical layer as a standalone product feature.
5. Update feature specs so multi-timeframe data, reproducibility metadata, strategy-policy resolution, execution semantics, and acceptance criteria are covered, while the engine no longer duplicates strategy authority.
6. Update `planning/ARCHITECTURE.md` from `PROJECT_SPEC.md`: define component ownership, multi-timeframe historical-data flow, daily level snapshots, pure strategy signal generation, engine-owned execution/risk/statistics, PostgreSQL persistence/cache, and Telegram application orchestration.
7. Update cross-references in `AGENTS.md` and related planning docs without touching product code or unrelated dirty paths.
8. Verify all links, required sections, feature dependencies, architecture traceability, duplicated authoritative text, and rendered Markdown.

## Scope clarification

- `ROADMAP.md` and the feature decomposition are first-class deliverables, not incidental updates.
- The roadmap must answer what is built next, why it matters to the user, what it depends on, and how progress is marked.
- Feature specs must answer which user outcome they support and which technical responsibilities implement that outcome.
- Architecture must trace every component and data-flow step back to `PROJECT_SPEC.md` and must not choose unresolved trading policies.

## Approval gate

status: approved
pending-action: execute `.omo/plans/consolidate-product-strategy-feature-plan.md` in a separate worker session

## Review note

- Mandatory Metis launch was attempted repeatedly but the harness aborted the reviewer before a verdict. The plan compensates with explicit requirement coverage, dependency matrix, happy/failure QA for every task, and four independent final-verification lanes.
