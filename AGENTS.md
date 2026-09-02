# AGENTS.md

## Purpose
This repository is a minimal MVP for a Telegram-based Bybit backtester.

## Working rules for OMO sessions
- Read this file at the start of every session.
- Prefer the existing MVP scope over broad product additions.
- Keep changes small, explicit, and reversible.
- Do not apply any code, documentation, or configuration changes until the user explicitly approves them.
- Do not add Web UI, WebSocket, Redis, Kubernetes, ML, or multi-strategy support unless the user explicitly requests it.
- Use Python, PostgreSQL, Docker, Docker Compose, Telegram Bot API, and `python-telegram-bot` as the default stack.
- The system must use real public Bybit market data and must not require trading API keys.
- Every code, configuration, or documentation change must update all affected technical documentation in both `docs/` and `planning/`.
- При любом изменении кода, конфигурации или документации обязательно обновляй всю затронутую техническую документацию одновременно в `docs/` и `planning/`.
- If a change affects user-facing behavior or scope, update `planning/PRODUCT.md` first, then update the implementation and every other affected document.
- Если изменение влияет на пользовательское поведение или scope, сначала обнови `planning/PRODUCT.md`, затем реализацию и все остальные затронутые документы.
- If a change touches architecture, update `planning/ARCHITECTURE.md` and every other affected document.
- Если изменение затрагивает архитектуру, обнови `planning/ARCHITECTURE.md` и все остальные затронутые документы.
- В папке `planning/` все документы и дальнейшие правки веди на русском языке.

## MVP behavior
- The bot should support `/backtest`, `/strategy`, `/status`, and `/help`.
- `/backtest BTCUSDT <period>` should accept only the fixed MVP symbol and a period, fetch missing D1/H1/M5 candles, store them locally, run one strategy, simulate trades, and return statistics in Telegram.
