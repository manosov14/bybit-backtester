# AGENTS.md

## Purpose
This repository is a minimal MVP for a Telegram-based Bybit backtester.

## Working rules for OMO sessions
- Read this file at the start of every session.
- Prefer the existing MVP scope over broad product additions.
- Keep changes small, explicit, and reversible.
- Do not add Web UI, WebSocket, Redis, Kubernetes, ML, or multi-strategy support unless the user explicitly requests it.
- Use Python, PostgreSQL, Docker, Docker Compose, Telegram Bot API, and `python-telegram-bot` as the default stack.
- The system must use real public Bybit market data and must not require trading API keys.
- If a change touches architecture, update `ARCHITECTURE.md` and `PRODUCT.md` when needed.
- If a change affects user-facing behavior or scope, update `PRODUCT.md` first, then code.

## MVP behavior
- The bot should support `/backtest`, `/strategy`, `/status`, and `/help`.
- `/backtest` should fetch missing historical candles, store them locally, run one strategy, simulate trades, and return statistics in Telegram.
