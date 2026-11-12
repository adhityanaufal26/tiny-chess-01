# tiny-chess-01

A 4 KB Python chess engine for the make.fun tiny-chess-01 bounty.

`main.py` is the whole engine: a single file, at most 4,096 bytes, Python 3.13
standard library only. It reads, imports and execs no other file in this
repository. The engine source is stored inside `main.py` as a compressed
(lzma + base85) payload; the first line unpacks and execs it. The unpacker
and the payload together are 4,081 bytes.

## Engine

- 0x88 board, make/unmake with incremental Zobrist hashing (strict
  en-passant key, matching the env: the key is set only when an en-passant
  capture is actually possible).
- Search: principal-variation search with a transposition table, null-move
  pruning, late-move reductions, killer moves, check extensions, and
  quiescence search (captures, promotions, check evasions, delta pruning).
- Evaluation: tapered piece-square tables (middlegame), bishop-pair bonus,
  and a formula king-centralisation term that grows as material comes off.
- Move ordering: transposition-table move, MVV-LVA captures, promotions,
  killers.
- Determinism: a fixed node budget (100,000 nodes per move) — no clock, no
  unseeded randomness, no dict-order dependence. Replaying the same trial
  produces byte-identical transcripts.

## Files

- `main.py` — the engine (packed, 4,081 bytes).
- `LICENSE` — MIT.
- `.makefun/owner.json` — claim file naming the entry wallet.
