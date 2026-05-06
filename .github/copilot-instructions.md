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

### Data Flow

1. User loads `/` → creates session cookie → displays home screen
2. User clicks "Start Game" → POST `/start` calls `GameSession.start_game()` → generates random board
3. User clicks square → POST `/toggle/{id}` → toggles marked state → checks for bingo
4. If bingo detected → shows modal, sets `game_state = GameState.BINGO`
5. User dismisses modal → POST `/dismiss-modal` → keeps board visible but hides modal

### Session Persistence

Sessions are **cookie-based** (Starlette middleware) with a signed session ID stored in `_sessions` dict. No database. Each session holds complete game state as a `GameSession` object.

## Key Patterns

### Board Generation

```python
# 5×5 grid, 25 squares total. Center (index 12) is always FREE SPACE (pre-marked).
# Remaining 24 squares get random questions from QUESTIONS list.
generate_board() -> list[BingoSquareData]  # Returns immutable square objects
```

### Winning Detection

```python
# Cached function computes all 12 possible winning lines:
# 5 rows + 5 columns + 2 diagonals
check_bingo(board) -> BingoLine | None
```

### HTMX Integration

Routes return partial HTML snippets (not full pages). HTMX swaps them into the DOM. Templates in `app/templates/components/` (bingo_board, game_screen, start_screen, bingo_modal). Base layout in `base.html`.

### Immutable Board Squares

`BingoSquareData` is frozen (immutable). Toggle creates new square with `model_copy(update=...)`. Benefits: predictable state, easier testing, no accidental mutations.

## Development Workflow

### Required Commands

```bash
# Run dev server (auto-reload on file change)
uv run uvicorn app.main:app --reload --port 8000

# Run tests (pytest discovers tests/ folder)
uv run pytest

# Lint code (ruff enforces E, F, I, N, W rules)
uv run ruff check .
```

### Testing Approach

- **Unit tests** (`test_game_logic.py`): Board generation, toggling, bingo detection
- **Integration tests** (`test_api.py`): HTTP endpoints using FastAPI TestClient. Must establish session before toggling (POST `/` first)

Example: `client.get("/")` sets session cookie, then `client.post("/toggle/0")` works.

### Commit Checklist

- [ ] `uv run ruff check .` passes (no lint errors)
- [ ] `uv run pytest` passes (all tests green)
- [ ] Code uses snake_case, type hints, no unused imports

## Styling Conventions

Custom CSS utility classes (Tailwind-like syntax) in `app/static/css/app.css`:

- **Layout**: `.flex`, `.flex-col`, `.grid`, `.grid-cols-5`, `.items-center`, `.justify-center`
- **Spacing**: `.p-4` (padding), `.mb-2` (margin-bottom), `.mx-auto` (horizontal centering)
- **Colors**: `.bg-accent` (#2563eb blue), `.bg-marked` (#dcfce7 green), `.text-gray-700`
- **Sizing**: `.w-full`, `.max-w-md`, `.text-lg`, `.text-5xl`

See `.github/instructions/css-utilities.instructions.md` for extended reference.

## Dependencies

- **fastapi**: Web framework with automatic OpenAPI docs
- **uvicorn**: ASGI server
- **jinja2**: Template engine (`.html` files in `templates/`)
- **itsdangerous**: Signing cookies (not currently used but imported)
- **httpx**: HTTP client for TestClient
- **ruff**: Fast linter + formatter
- **pytest**: Testing framework

Python 3.13+ required.

## Project Structure

```
app/
├── main.py              # FastAPI app + routes
├── game_logic.py        # Board generation, bingo detection
├── game_service.py      # Session management
├── models.py            # Pydantic models
├── data.py              # Question bank
├── templates/           # Jinja2 HTML templates
│   ├── base.html
│   ├── home.html
│   └── components/      # Partial templates for HTMX swaps
├── static/
│   ├── css/app.css      # Utility classes
│   └── js/htmx.min.js
tests/
├── test_api.py          # API/integration tests
└── test_game_logic.py   # Game logic unit tests
```

## Common Tasks

**Add a new question**: Edit `app/data.py` → `QUESTIONS` list (must be exactly 24).

**Modify board size**: Change `BOARD_SIZE = 5` in `game_logic.py`, update `_get_winning_lines()`, adjust CSS grid.

**Debug session state**: Use `app.game_service._sessions` dict. Each key is session ID, value is `GameSession`.

**Style a template**: Use CSS classes from `app.css`. All custom utilities follow Tailwind naming.

**Add a route**: In `main.py`, call `_get_game_session(request)`, modify session state, return template response.

## Avoid

- Don't mutate board squares directly. Use `model_copy()` instead.
- Don't add business logic to templates (keep it in `game_service.py` or `game_logic.py`).
- Don't mix cookie logic with route logic; always call `_get_game_session()`.
