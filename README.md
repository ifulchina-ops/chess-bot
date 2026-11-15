# Tiny Chess Engine — 4 KB of Python

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/ifulchina-ops/chess-bot/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/ifulchina-ops/chess-bot/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in a single `main.py` of 4,079 bytes, written for the
make.fun bounty "Build the strongest chess engine in 4 KB of Python".
Original work — a compact negamax searcher with a 0x88 board, built from
scratch, not a copy of sunfish or any other engine.

## Design

- **Board:** 0x88, signed pieces; incremental material + piece-square eval
  and Zobrist hash update on make/unmake (fixed-seed generator).
- **Rules:** full legal move generation — castling, en passant, promotion,
  check/mate/stalemate; third repetition scores 0 in search.
- **Search:** negamax PVS + LMR + null-move pruning (R=2), iterative
  deepening to a fixed budget of 100,000 nodes per move (never a clock —
  replay is deterministic). Quiescence over captures, promotions, and all
  evasions in check.
- **Ordering:** transposition-table move first, MVV-LVA captures, two
  killer moves per ply, promotion bonus. TT with mate-distance adjustment.
- **Speed:** make() skips the legality test unless in check, king move, en
  passant, or the mover is aligned with its own king; 0x88→64 lookup table.
- **Eval:** material (P100 N320 B330 R500 Q900) + piece-square tables
  (192 values, left-right symmetric) + 30-point bishop-pair bonus.
- **Root safety:** only the environment's own `legal_moves` are ever played;
  the engine resigns when its root score falls below −1500.

## Packing

The readable source (~18 KB) is minified (comments/docstrings/prints
stripped, identifiers renamed by frequency, tabs + minimal spacing), then
lzma-compressed and base85-encoded into a one-line stub. The piece-square
table lives in the stub as plain text (it does not compress).

## Trial result

House gauntlet, 16 games at 100,000 nodes/move: **5 wins, 4 draws,
7 losses, Elo 1326**. Head-to-head vs the previous entry (R6, 50k nodes):
**7 wins, 9 draws, 0 losses** (+163 Elo) over 16 games from book openings.

## Run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

Python 3.13, standard library only. The engine talks to the make.fun
environment over HTTP: it asks for the game in play, sends moves in UCI,
and plays until the trial reports done.

## License

MIT — see LICENSE.
