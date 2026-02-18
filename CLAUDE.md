# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A CLI chess engine written in the Icon programming language. The engine supports human vs. computer play and computer self-training mode.

## Build and Run

Requires the Icon compiler (`icont`). To compile with a specific brain module:

```bash
icont -o chess chess.icn brain.icn
icont -o chess chess.icn brainless.icn
```

The `link brain` statement in chess.icn determines which brain module is linked. The file linked must match one of the brain modules (brain.icn or brainless.icn).

Run modes:
- `./chess` — human plays white vs. computer
- `./chess -b` — computer plays white vs. human
- `./chess -c N` — computer trains against itself for N games

Human moves are entered as coordinate pairs (e.g., `e2e4`). Pawn promotion appends the piece letter (e.g., `e7e8Q`). Type `draw` to claim a draw.

## Architecture

**chess.icn** — Core engine containing all game logic:
- Board representation: 8x8 array of `squarecontent(squarecolor, squarepiece)` records
- Piece tracking: `pieces` is a list of 6 piece types, each containing sets of `position(file, rank)` for white and black
- Game state flags: `flagrecord(cancastle, enpassant, draw, promotion, origin, dest, counter)` — castling rights use a bitmask where `a=1`, `h=8`, `a+h=9`
- Move generation per piece type (`generateking`, `generaterook`, etc.) with legality checking via `testmove` which copies the board, applies the move, and verifies king safety
- The `check` parameter controls whether move generation validates for check (pass `king` for legal moves, `&null` for attack squares only)

**brain.icn** — AI module with a `getcomputermove` procedure and `initializebrain`/`chessfunction` helpers. Currently picks a random piece that has legal moves and plays a random legal move.

**brainless.icn** — Minimal AI module: same random move strategy as brain.icn but without the brain/learning scaffolding.

**chessless.icn** — Variant of chess.icn that links `brainless` instead of `brain`.

## Key Conventions

- Colors: `white=1`, `black=2`, `empty=0`. `othercolor` uses `3 - color`.
- Files a-h map to integers 1-8. Ranks are integers 1-8.
- Pieces: `king=1` through `pawn=6`.
- Back rank computed as `color^3` (white=1, black=8).
- Draw detected at 100 half-moves (50-move rule).
- All source files use tabs for indentation and Icon's semicolon-terminated style.
