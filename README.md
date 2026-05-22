# Reversi / Othello AI

A Python simulator for [Reversi](https://en.wikipedia.org/wiki/Reversi) (Othello) with pluggable AI agents. Play head-to-head, watch games with a visual board, or batch many matches to compare win rates and move times.

The main competitive agent (`student_agent`) uses **iterative-deepening minimax** with alpha–beta pruning, move ordering, transposition caching, and position heuristics (corners, mobility, center control, and related patterns).

## Requirements

- Python 3.10+ (the student agent uses `match` / `case`)
- Dependencies in `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Quick start

Run a single game (default: two random agents on a random even board size between 6 and 12):

```bash
python simulator.py
```

Watch your agent play with the UI:

```bash
python simulator.py --player_1 student_agent --player_2 greedy_agent --display
```

Play as a human against the student agent on an 8×8 board:

```bash
python simulator.py --player_1 human_agent --player_2 student_agent --board_size 8 --display
```

Compare two agents over many games (win percentage and max turn time):

```bash
python simulator.py --player_1 student_agent --player_2 mcts_agent --autoplay --autoplay_runs 20
```

## Command-line options

| Flag | Default | Description |
|------|---------|-------------|
| `--player_1` | `random_agent` | Registered name for Blue (Player 1) |
| `--player_2` | `random_agent` | Registered name for Brown (Player 2) |
| `--board_size` | random | Fixed board size (6, 8, 10, or 12) |
| `--board_size_min` | 6 | Minimum size in autoplay |
| `--board_size_max` | 12 | Maximum size in autoplay |
| `--display` | off | Show matplotlib UI |
| `--display_delay` | 0.4 | Seconds between frames |
| `--display_save` | off | Save board images to disk |
| `--display_save_path` | `plots/` | Output directory for saved frames |
| `--autoplay` | off | Run many games and report statistics |
| `--autoplay_runs` | 10 | Number of games in autoplay |

In autoplay, board sizes are chosen randomly from even sizes in `[board_size_min, board_size_max]`, and players swap colors across runs for fairness.

## Built-in agents

| Name | Description |
|------|-------------|
| `random_agent` | Uniform random legal move |
| `human_agent` | Interactive play via the UI |
| `greedy_agent` | Heuristic / search-based greedy play |
| `mcts_agent` | Monte Carlo tree search |
| `gpt_greedy_corners_agent` | Corner-focused greedy strategy |
| `student_agent` | Iterative-deepening minimax with custom evaluation |

Agents are registered with `@register_agent("name")` in `store.py`. Import your module in `agents/__init__.py` so it loads at startup.

## Game representation

- **Board**: `numpy` array of shape `(n, n)` with `0` = empty, `1` = Blue, `2` = Brown.
- **Move**: `(row, col)` tuple, zero-indexed.
- **API**: Each agent implements `step(board, player, opponent)` and returns a legal move.

Shared rules and utilities live in `helpers.py` (`get_valid_moves`, `execute_move`, `check_endgame`, `count_capture`, etc.). Invalid or failing moves fall back to a random legal move.

## Project layout

```
agents/           # Agent implementations (subclass Agent)
  agent.py        # Base agent interface
  student_agent.py
  mcts_agent.py
  ...
helpers.py        # Game rules and board utilities
world.py          # Game loop, turns, timing, UI hookup
simulator.py      # CLI entry point and autoplay
ui.py             # Matplotlib board display
store.py          # Agent registry decorator
constants.py      # Player IDs, colors, board limits
```

## Writing your own agent

1. Subclass `Agent` in `agents/agent.py`.
2. Decorate the class with `@register_agent("my_agent")`.
3. Implement `step(self, board, player, opponent)` using helpers from `helpers.py`.
4. Import the module in `agents/__init__.py`.

Example skeleton:

```python
from agents.agent import Agent
from store import register_agent
from helpers import get_valid_moves

@register_agent("my_agent")
class MyAgent(Agent):
    def __init__(self):
        super().__init__()
        self.name = "MyAgent"

    def step(self, board, player, opponent):
        moves = get_valid_moves(board, player)
        return moves[0]  # replace with your logic
```

## Student agent strategy (overview)

`student_agent` searches with increasing depth until a per-board time budget is reached:

- **Search**: Minimax with alpha–beta pruning and a transposition table.
- **Ordering**: Moves sorted by a quick static evaluation before expansion.
- **Evaluation**: Weighted mix of corner control, mobility, center squares, capture potential, and endgame scoring.

Time limits are tuned per board size (roughly ~1.9s per move on 6×6 through 12×12).

## License

See repository history or course materials for license terms if this project was submitted as coursework.
