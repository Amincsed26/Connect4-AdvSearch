# 🔴🟡 Connect-Four

Web-based Connect Four game with a Flask backend that presents three different game-tree search algorithms — **MinMax**, **Alpha-Beta Pruning**, and **Expectiminimax** — against a 6×7 board, plus a benchmarking suite comparing their performance.

## ✨ Features

### 🎮 Gameplay
- 6-row × 7-column Connect Four board (`'r'` red, `'y'` yellow, `''` empty cell)
- AI move suggestion via a REST API, selectable per request between three search strategies
- Board scoring endpoint to count already-connected four-in-a-rows for a given piece
- Search-tree inspection endpoint (returns the last AI computation as a tree of node values)

### 🤖 AI Search Algorithms
- **MinMax** — exhaustive game-tree search with a transposition-style board cache to avoid re-evaluating repeated positions
- **Alpha-Beta Pruning** — MinMax with alpha/beta cutoffs to prune branches that can't affect the final decision
- **Expectiminimax** — models an imperfect opponent: instead of always dropping in the intended column, the "chance" layer distributes probability across the intended column and its neighbors (e.g. 60% intended column, 20%/20% adjacent columns, with edge-case rebalancing near the board edges)

### 🧮 Heuristic Evaluation
Board positions are scored using a weighted combination of:
| Criterion | Weight |
|---|---|
| Already-connected four-in-a-rows | 2000 |
| Three-in-a-row that could become four | 1000 |
| Two-in-a-row that could become four | 150 |
| Pieces in the middle column | 20 |
| Pieces in the two columns next to the middle | 10 |
| Pieces in remaining columns (excluding corners) | 5 |
| Pieces in the board corners | 1 |

Each criterion is scored for both the current player and the opponent, and the heuristic returns the difference. A full board instead scores purely on already-connected fours, scaled by 2000.

### 📊 Performance Benchmarking
`Performance.py` runs MinMax and Alpha-Beta head-to-head across increasing search depths on randomly generated mid-game boards, recording:
- Average time per depth (ms)
- Average nodes expanded per depth

Results are written to `normal.txt` / `pruning.txt` and plotted with matplotlib (time vs. depth, nodes visited vs. depth, and direct MinMax-vs-Alpha-Beta comparisons) — these are the `Figure_*.png` charts in the repo.

---

## 📂 Project Structure
```
Application/
├── App.py                              # Flask entry point, serves index.html and registers routes
├── bonus.py                            # Standalone script to run Alpha-Beta on a sample board
├── Performance.py                      # Benchmarks MinMax vs Alpha-Beta and plots results
├── normal.txt / pruning.txt            # Saved benchmark timings and node counts
├── Controllers/
│   ├── Controller.py                   # Flask blueprint: /score, /ai, /tree routes
│   └── ControllerValidator.py          # Request payload validation
├── Modules/
│   ├── AIModule.py                     # Dispatches to the selected solver (AlphaBeta/MinMax/ExpectiiMinMax)
│   └── ScoreModule.py                  # Board scoring (connected-fours count)
└── Services/
    ├── AlphaBetaService.py             # Alpha-beta pruning solver
    ├── MinMaxService.py                # Plain minimax solver
    ├── ExpectiMiniMaxService.py        # Expectiminimax solver (probabilistic opponent)
    ├── GameService.py                  # Board utilities: valid moves, insert piece, scoring helpers
    ├── Heuristic.py                    # Weighted heuristic evaluation
    ├── Node.py                         # Game-tree node (value, best child, children, nodes expanded)
    ├── Solver.py                       # Abstract base class for solvers
    └── HeuristicCriterias/
        ├── AlreadyConnectedFours.py    # Counts existing four-in-a-rows
        ├── CouldConnectFourInOneMove.py# Counts open three-in-a-rows and two-in-a-rows one move from four
        ├── CouldConnectFourInTwoMoves.py# Counts two-in-a-rows two moves from four
        └── SinglePiecePosition.py      # Positional scoring (middle column, corners, etc.)
```

---

## 📡 REST API

| Method | Endpoint | Body | Description |
|---------|----------|------|-------------|
| POST | `/score` | `{ "board": [[...]], "piece": "r" \| "y" }` | Returns the number of already-connected four-in-a-rows for `piece` |
| POST | `/ai` | `{ "board": [[...]], "piece": "r" \| "y", "max_depth": int, "method": "AlphaBeta" \| "MinMax" \| "ExpectiiMinMax" }` | Returns the AI's chosen column |
| GET | `/tree` | — | Returns the search tree (value, best child, children) from the most recent `/ai` call |

---

## 🛠️ Tech Stack
- Python, Flask (backend/API)
- Jinja templates (`index.html`) for the web page served by Flask
- JavaScript/CSS/HTML front end for the interactive board
- matplotlib (performance benchmarking/plots)

---

## 🚀 Getting Started

### Prerequisites
- Python 3.x
- Flask
- matplotlib (only needed to run `Performance.py`)

### Installation
```bash
git clone https://github.com/Amincsed26/Connect-Four.git
cd Connect-Four/Application
pip install flask matplotlib
```

### Run the app
```bash
python App.py
```
Then open the app in your browser (Flask's default dev server runs on `http://127.0.0.1:5000`).

### Run the benchmark
```bash
python Performance.py
```
This generates `normal.txt`, `pruning.txt`, and the comparison plots.
