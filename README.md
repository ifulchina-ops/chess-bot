# Tiny Chess Engine (4 KB)

A chess engine in a single `main.py` (4,071 bytes), built for the make.fun
"Build the strongest chess engine in 4 KB of Python" bounty. Original code —
not a copy of sunfish or any other engine.

## Run

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:PORT python3 main.py
```

Python 3.13, stdlib only (`json`, `os`, `random`, `urllib`). The engine talks
to the make.fun environment over HTTP: it asks for the game in play, sends
moves in UCI, and plays until the trial reports done. No other file is read.

## Design

- **Board:** 0x88, signed pieces, incremental material + piece-square eval and
  incremental Zobrist hash (fixed-seed generator, deterministic).
- **Rules:** full legal move generation — castling, en passant, promotion,
  check / checkmate / stalemate; third-repetition and fifty-move draws are
  scored 0 in search.
- **Search:** negamax alpha-beta with iterative deepening to a **fixed budget
  of 150,000 nodes per move** (never a clock — replay is deterministic),
  quiescence over captures, promotions, and all evasions when in check.
  Null-move pruning (R=2) when not in check and beta is not a mate score.
- **Ordering:** transposition-table move first, MVV-LVA captures, two killer
  moves per ply, promotion bonus. TT with mate-distance adjustment, cleared
  deterministically above 250,000 entries.
- **Eval:** material (P100 N320 B330 R500 Q900) + Michniewski piece-square
  tables + bishop-pair bonus (30).
- **Root safety:** only the environment's own `legal_moves` are ever played,
  so a move-generation disagreement can never produce an illegal move.

## Packing

The readable source (~16.6 KB) is minified (comments/docstrings/prints
stripped, identifiers renamed by frequency, tabs + minimal spacing), then
lzma-compressed and base85-encoded into a one-line stub. The high-entropy
piece-square table payload lives in the stub as plain text (it does not
compress). See `build-transcript.md` for validation results and benchmarks.

## License

MIT — see LICENSE.
