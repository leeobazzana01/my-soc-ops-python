# Copilot Instructions: Soc Ops

## ✅ Mandatory Development Checklist

Before committing **any** change:

```bash
uv run ruff check .     # Lint (no errors allowed)
uv run pytest           # Tests (all must pass)
uv run uvicorn app.main:app --reload --port 8000  # Run locally to verify
```

## Project Overview

**Soc Ops** is a Social Bingo game (FastAPI + Jinja2 + HTMX). Players mark a 5×5 board to find people matching icebreaker questions—get 5 in a row to win.

## Architecture

- **`app/main.py`**: FastAPI routes + HTMX endpoints (home, start, toggle, modal, reset). All return partial HTML for HTMX swaps.
- **`app/game_logic.py`**: Pure logic—board generation, bingo detection (5 rows + 5 columns + 2 diagonals), square toggling. Uses cached `_get_winning_lines()`.
- **`app/game_service.py`**: `GameSession` dataclass manages state (board, game_state, winning_line, modal visibility). In-memory storage by session ID.
- **`app/models.py`**: Pydantic models—`GameState` enum, frozen `BingoSquareData`, `BingoLine`.
- **`app/data.py`**: 24 questions + `FREE_SPACE` constant (center square).

## Key Patterns

**Board**: 5×5 grid (25 squares). Center (index 12) always FREE SPACE. Remaining 24 get random questions from `QUESTIONS` list.

**Winning Detection**: Cached function checks all 12 possible lines (5 rows + 5 columns + 2 diagonals).

**HTMX**: Routes return partial HTML (not full pages). Templates in `app/templates/components/`. HTMX swaps them into DOM.

**Immutable Squares**: `BingoSquareData` is frozen. Toggle creates new square with `model_copy(update=...)`. Prevents accidental mutations.

**Sessions**: Cookie-based via Starlette middleware. Session ID stored in `_sessions` dict (in-memory). Each session holds `GameSession` object.

## Common Tasks

- **Add question**: Edit `app/data.py` → `QUESTIONS` (must be exactly 24)
- **Change board size**: Update `BOARD_SIZE = 5` in `game_logic.py` + `_get_winning_lines()` + CSS grid
- **Debug session**: Check `app.game_service._sessions` dict (key = session ID, value = `GameSession`)
- **Style template**: Use `.bg-accent` (#2563eb), `.bg-marked` (#dcfce7), `.text-gray-700` from `app/static/css/app.css`
- **Add route**: Call `_get_game_session(request)`, modify state, return template response

## Design Guide — Cyberpunk Galaxy

Purpose: Keep the UI consistent with the Dark Mode Noir / Cyberpunk Galaxy redesign: neon accents, deep space backgrounds, subtle motion, and sci‑fi typography.

Principles
- Typography: use Orbitron for headings and Oxanium for UI text (fonts imported in app/static/css/app.css).
- Color & Theme: prefer CSS variables (e.g. --bg, --panel, --neon-cyan, --neon-pink, --neon-purple, --accent) defined in app/static/css/app.css.
- Motion: favor CSS-only effects (drift, gentle bounce, glow). Respect prefers-reduced-motion.
- Backgrounds: layer radial gradients + a low-opacity starfield; decorative planets use .planet-decor.
- Components: panels -> .panel-neon; primary buttons -> .btn-neon; badges -> .badge-neon; use .neon-text for headings.
- Assets: place SVG/PNG assets in app/static/img/ (spaceships, aliens, planets). Use small, optimized files and size with .spaceship.
- Accessibility: maintain sufficient contrast, use aria-labels, focus styles, and reduced-motion support.

Implementation notes
- When adding visuals, add a short CSS utility in app/static/css/app.css following existing patterns.
- Reuse variables for colors and glows; avoid hard-coded hexes in templates.
- Document any new assets or style variables in this file.

