# tiny-chess-01

A 4 KB Python chess engine for the make.fun tiny-chess-01 bounty.

`main.py` is the whole engine: a single file, at most 4,096 bytes, Python 3.13
standard library only. It reads, imports and execs no other file in this
repository. The engine source is stored inside `main.py` as a compressed
(lzma + base85) payload; the first line unpacks and execs it. The unpacker
and the payload together are 4,062 bytes.

## Engine

- 0x88 board, make/unmake with incremental Zobrist hashing (strict
  en-passant key, matching the env: the key is set only when an en-passant
  capture is actually possible).
- Search: principal-variation search with a transposition table, null-move
  pruning, late-move reductions, killer moves, check extensions, and
  quiescence search (captures, promotions, check evasions, delta pruning,
  plus a minimal static-exchange evaluation that skips a capture when the
  attacker is worth more than the victim and the target square is defended).
- Repetition detection: the search returns a draw score when the current
  position repeats a position already on the search path (path-only;
  game history is not tracked, so it cannot poison the table).
- Evaluation: tapered piece-square tables (middlegame), bishop-pair bonus,
  and a formula king-centralisation term that grows as material comes off.
- Move ordering: transposition-table move, MVV-LVA captures, promotions,
  killers.
- Determinism: a fixed node budget (150,000 nodes per move) — no clock, no
  unseeded randomness, no dict-order dependence. Replaying the same trial
  produces byte-identical transcripts.

## Changes vs v8 (the submitted entry, 1253 Elo)

- Minimal SEE in quiescence: at fixed depth it saves 7–16% of nodes
  (depth-5 mean 92,005 → 85,705; depth-6 mean 550,044 → 460,842), but a
  16-game local match at 20,000 nodes per move showed it is roughly Elo
  neutral (−22 Elo, within noise).
- Path-only repetition detection (bench-neutral on single positions, fires
  5,075 times in a bare-king search at the 150k budget).
- Node budget raised 100,000 → 150,000 (bought one extra ply on the
  italian bench position, depth 5 → 6).
- History table was tried and dropped: bench-identical to no history
  (a no-op), so it was removed to save bytes.

## Files

- `main.py` — the engine (packed, 4,062 bytes).
- `LICENSE` — MIT.
- `.makefun/owner.json` — claim file naming the entry wallet.
