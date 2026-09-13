# Ursatz-GUI — Status

**Last verified against repo state:** 2026-09-13, commit `877c52a` (PR #26). See
`Reconciliation_Log.md` for how this was checked.

## Purpose

A local web UI hosting music-theory analysis tools built on top of the `Ursatz` C library
(git submodule). Designed as a thin host/shell: a shared service layer wraps the only code
allowed to call into Ursatz's persistence/analysis functions, independent "tool modules" live
under their own `/api/<tool_name>/...` route namespaces and never call each other, and a
static frontend shell renders a sidebar/topbar around whichever tool is active. Gives
Ursatz's analysis engine a browser-based front end. Native packaging, authentication, and the
Generator tool remain explicitly out of scope.

## Current implementation

**Backend (`backend/src/`):** civetweb-based HTTP server (`server.c`), routes under
`/api/<tool>/...`:
- `health` — persistence round-trip check.
- `analyzer` — MIDI upload → import/analyze/save (`POST /api/analyzer/pieces`), list
  (`GET /api/analyzer/pieces`), detail (`GET /api/analyzer/pieces/<id>`), rename
  (`PATCH`, added PR #16), delete, and MusicXML export
  (`GET /api/analyzer/pieces/<id>/musicxml`, PR #20).
- `corpus` — Collections/Corpus tool, fully implemented (`routes/corpus.c`,
  `service/corpus_service.c`), not just a sidebar placeholder.

**Frontend (`frontend/public/`), three real views, all wired to their backend APIs:**
- **Library** (PR #17) — table view of saved pieces with rename/delete, upload via a
  drop-zone/file-input control, filename-click navigation into Analysis.
- **Collections** (PR #18) — a grid of cards (restyled from the old Corpus sidebar entry),
  detail view listing a collection's pieces with a remove-piece action.
- **Analysis** (PR #19) — tabbed view with per-category claim lists, a claim detail panel
  (Evidence/Provenance, Accept/Dispute present but disabled — no promotion endpoint yet), and
  a **Score tab** (PR #20) rendering the piece via a vendored OpenSheetMusicDisplay against
  Ursatz's exported MusicXML, with **note-level highlighting** (PR #21) tying selected claims
  to the notes they target.

**Service layer (`analysis_service.c`):** the sole caller of Ursatz's analyzer/persistence
functions. All 7 Framework-stage analyzer types (Cadence, Motif, Phrase, Period, Sentence,
Form, Thematic Return) are wired into the upload/analyze path and surfaced in piece-detail
responses (PR #13) — not just Key/Chord as in the original scaffold. Maintains two
ursatz-gui-owned SQLite tables alongside Ursatz's own per-piece database:
`ursatz_gui_piece_meta` (original filename, PR #12) and `ursatz_gui_relationships`
(Period/Sentence/Form/Thematic Return output — a stopgap pending a real Ursatz-side
`RelationshipRepository`, see `Ursatz-Library/Known_Gaps.md`).

## Fixed since the original scaffold (verified in code, not just claimed)

- **Piece-detail heap-overflow** (PR #10) — `handle_get_piece`'s buffer sizing underflowed
  once responses grew past a fixed estimate. Fixed with a growable `JsonBuffer`.
- **Provenance mislabeling** (PR #11) — frontend was inverting analyzed/hand-entered labels;
  `PROVENANCE_LABELS` corrected.
- **Original filename capture/display** (PR #12) — persisted and shown alongside piece IDs.
- **SQLite busy-timeout** (PR #42, upstream in Ursatz, bumped in here) — avoids immediate
  `SQLITE_BUSY` on concurrent open.
- **MusicXML export overlap detection / error-code propagation** (PRs #22–25).

## Build & test

CMake, C11 compiler, civetweb v1.16 (via `FetchContent`), sqlite3 (transitively via Ursatz).
`external/ursatz` submodule, currently at commit `5d10667...` (bumped repeatedly past the
Framework-stage pin as Ursatz has continued shipping — see `Ursatz-Library/Status.md`). No
Node/JS build tooling — frontend is hand-written static HTML/CSS/JS plus a vendored OSMD
UMD build.

## What depends on this repo

Nothing — this is a leaf application.
