# tiny-chess-01

A 4 KB Python chess engine for the make.fun tiny-chess-01 bounty.

`main.py` is the whole engine: a single file, at most 4,096 bytes, Python 3.13
standard library only. It reads, imports and execs no other file in this
repository. The engine source is stored inside `main.py` as a compressed
(lzma + base85) payload; the first line unpacks and execs it. The unpacker
and the payload together are 4,076 bytes.

## Engine (v11, from scratch)

- 0x88 board, make/unmake with incremental Zobrist hashing (strict
  en-passant key, matching the env: the key is set only when an en-passant
  capture is actually possible) and an **incremental evaluation** that is
  maintained in make/unmake, so the static evaluation is O(1) instead of a
  full-board scan on every node.
- Search: principal-variation search with a transposition table, null-move
  pruning, late-move reductions, killer moves, check extensions, and
  quiescence search (captures, promotions, check evasions, delta pruning,
  plus a minimal static-exchange evaluation that skips a capture when the
  attacker is worth more than the victim and the target square is
  defended). Futility pruning in the main search (depth <= 2).
- Repetition detection: the search returns a draw score when the current
  position repeats a position already on the search path (path-only;
  game history is not tracked, so it cannot poison the table).
- Evaluation: combined material + piece-square table (CPW-like values),
  kept as a single incremental score. No bishop-pair bonus, no king
  centralisation term (both were removed to fit the 4,096-byte cap; the
  bench showed removing them changed node counts by ~1%).
- Move ordering: transposition-table move, then a value-based order
  (piece-square delta + capture value + promotion gain) with killer-move
  bonuses — sunfish-style, instead of the MVV-LVA used by v10h.
- Determinism: a fixed node budget (150,000 nodes per move) — no clock, no
  unseeded randomness, no dict-order dependence. Replaying the same trial
  produces byte-identical transcripts.

## Changes vs v10h (the submitted entry, 1362 Elo)

- Incremental evaluation (O(1) `ev()`) instead of a full-board scan:
  this is the main win.
- Value-based move ordering instead of MVV-LVA.
- Fixed-depth bench (identical positions): depth-5 mean 85,705 nodes
  (v10h) -> 44,485 nodes (v11); depth-6 mean 460,842 -> 153,939.
  v11 uses roughly 2x fewer nodes at depth 5 and 3x fewer at depth 6,
  so at the same 150,000-node budget it searches about 2-3 extra plies
  deeper than v10h.
- Bishop-pair bonus and king-centralisation term were removed to fit the
  byte cap; node counts were unchanged (depth-6 mean 151,574 -> 153,939).

## Files

- `main.py` — the engine (packed, 4,076 bytes).
- `LICENSE` — MIT.
- `.makefun/owner.json` — claim file naming the entry wallet.
