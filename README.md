🌐 [Português (BR)](README.pt_BR.md) | [Español](README.es.md)

# Soc Ops — Social Bingo with Personality

Soc Ops turns icebreakers into a playful game: a 5×5 bingo board of human-first questions. Talk, mark, and connect—get five in a row to win. Perfect for meetups, orientation events, and team mixers.

![Playful board illustration](docs/board-placeholder.png)

Why this project matters
- Encourages real conversation over small-talk
- HTMX + FastAPI for a fast, accessible frontend experience
- Tiny, testable core logic so it’s easy to extend or embed

---

## Quick Start — Play locally (30s)

1. Create a virtualenv and activate it (optional but recommended)

   python -m venv .venv && source .venv/bin/activate

2. Install dependencies

   pip install -r requirements.txt

3. Run checks and tests (project conventions)

   uv run ruff check .
   uv run pytest

4. Start the server

   uv run uvicorn app.main:app --reload --port 8000

Open http://localhost:8000 and start a game.

---

## Highlights

- Lightweight FastAPI backend + Jinja2 templates
- HTMX endpoints for partial swaps (no heavy SPA required)
- Pure, unit-testable game logic in app/game_logic.py
- In-memory session-backed GameSession for quick demos

---

## How it works (tl;dr)
- A 5×5 board is generated; the center square is a FREE SPACE.
- 24 curated questions fill the remaining squares.
- Players click to mark squares; the server evaluates winning lines (rows, columns, diagonals).
- Templates in app/templates/components/ render HTMX fragments for live updates.

---

## Contribute
- Want to add questions, a theme, or mobile polish? Contributions welcome.
- Follow the dev checklist in the repo: run ruff, pytest, then run the app locally to validate changes.
- Edit app/data.py to update QUESTIONS (keep it at 24 items) or extend GameSession behavior in app/game_service.py.

---

## Resources
- Workshop & lab guide: [`workshop/`](workshop/)
- Code overview: app/main.py, app/game_logic.py, app/game_service.py, app/data.py

---

Made with ❤️ for better small talk — enjoy connecting.
