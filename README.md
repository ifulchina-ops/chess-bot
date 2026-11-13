# Tiny Chess Engine — 4 KB of Python

A chess engine in a single `main.py` of 4,072 bytes, written for the
make.fun bounty "Build the strongest chess engine in 4 KB of Python".
Original work — a compact negamax searcher with a 0x88 board, built from
scratch, not a copy of sunfish or any other engine.

## Design

### Board and state

- **0x88 board:** signed pieces on a 128-square array, so off-board
  tests are one bit check (`sq & 0x88`) instead of bounds logic.
- **Incremental state:** eval and Zobrist hash update on make/unmake,
  so the search never rescans the board; random-game checks recompute
  both from scratch to catch drift.
- **Full rules:** castling, en passant, promotion, check, mate and
  stalemate. Third repetition scores 0 inside the search.

### Search

- **Negamax alpha-beta, iterative deepening** to a fixed budget of
  150,000 nodes per move — node-based, never time-based, so games replay
  deterministically from a fresh process.
- **Quiescence** covers captures, promotions, and all evasions in
  check; stand-pat uses the incremental eval plus the bishop-pair bonus.
- **Null-move pruning (R=2)** applies out of check only; positions in
  check are searched fully.
- **Move ordering:** transposition-table move first, then MVV-LVA
  captures, two killer moves per ply, and a promotion bonus.
- **Transposition table** keyed by Zobrist hash, mate scores adjusted
  by ply; it clears past 250,000 entries, a bound set by the search
  history, so determinism holds.

### Evaluation

Material (pawn 100, knight 320, bishop 330, rook 500, queen 900) plus
piece-square tables and a 30-point bishop-pair bonus. The six tables
are left-right symmetric, so only the a-d half of each row is stored
(192 values total), base85-encoded. Black mirrors with index^56.

### Root safety

Only moves from the environment's own `legal_moves` list are ever
played, so the engine can never output an illegal move — even if the
search disagrees with the environment about a position.

## Packing

The readable source is about 16.6 KB. It is minified (comments and
docstrings stripped, identifiers renamed, minimal whitespace), then
lzma-compressed and base85-encoded into a two-line `main.py`. The
packed artifact is verified on its own: perft counts, mate puzzles,
determinism and eval/hash consistency all check out on the packed
file itself.

## Run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

Python 3.13, standard library only. The engine talks to the make.fun
environment over HTTP: it asks for the game in play, sends moves in UCI,
and plays until the trial reports the game done.

## Trial result

House gauntlet, 16 games: **6 wins, 6 draws, 4 losses, elo 1464** —
above the sunfish bar for the trial. Locally it scored 7.5/8 against
sunfish at 50k nodes a move.

## Development

Built in agent sessions whose transcripts ship with the entry: the
engine, the packer and this README were written and revised in-session.

## License

MIT — see LICENSE.
