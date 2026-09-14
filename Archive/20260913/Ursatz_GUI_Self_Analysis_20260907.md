# Ursatz-GUI — Repository Analysis

## 1. Purpose

Ursatz-GUI is a local web UI that hosts music-theory analysis tools built on top of the separate `Ursatz` C library (a git submodule). It is explicitly designed as a thin host/shell rather than a monolith: a shared service layer wraps the only code allowed to call into Ursatz's persistence/analysis functions, independent "tool modules" (currently an Analyzer, with Corpus and Generator planned) live under their own `/api/<tool_name>/...` route namespaces and never call each other, and a minimal static frontend shell renders a sidebar/topbar around whichever tool is active. It exists to give Ursatz — a music analysis engine (MIDI import, key estimation, chord identification, interpretation persistence) — a browser-based front end, while keeping native packaging, authentication, and the Generator tool's implementation explicitly out of scope for now.

## 2. Public API surface

### Process entry point
- `int main(void)` — `backend/src/main.c:29`. Installs SIGINT/SIGTERM handlers, starts the server on a hardcoded port, idles until signaled, then stops the server.

### Server lifecycle (`backend/src/server.h`)
- `struct mg_context *ursatz_gui_server_start(const char *listen_port, const char *frontend_dir)` — starts civetweb, sets `document_root` to `frontend_dir`, 4 worker threads, registers all tool routes.
- `void ursatz_gui_server_stop(struct mg_context *ctx)` — shuts the server down.

### HTTP endpoints
- `GET /api/health/status` — reference/health-check tool (`backend/src/routes/health.c:32`). Runs a persistence open/close check against `ursatz_gui_health_check.db`; returns `{"ok": bool, "message": str}`.
- `POST /api/analyzer/pieces` — `backend/src/routes/analyzer.c` (`handle_post`, line 64). Body is raw MIDI bytes (max 64MB). Imports and analyzes the piece, saves it, returns `{"ok":true,"id":"<piece-id>"}` or `{"ok":false,"message":...}`.
- `GET /api/analyzer/pieces` — `analyzer.c` (`handle_get`, line 93). Lists saved piece IDs: `{"ok":true,"pieces":[{"id":...},...]}`.
- `GET /api/analyzer/pieces/<id>` — `analyzer.c` (`handle_get_piece`, line 132), dispatched via a manual `pieces_handler` that distinguishes the exact `/pieces` path from a `/pieces/<id>` subpath because civetweb tries exact- then prefix-match before wildcard registrations. Returns note count and interpretation summaries, or `{"ok":false,"message":"piece not found"}` (HTTP 200 by this API's own convention). Piece IDs are validated against an alnum/`-`/`_` whitelist to block path traversal.

### Extension point (not itself an endpoint)
- `typedef void (*ursatz_gui_tool_register_fn)(struct mg_context *ctx)` — `backend/include/ursatz_gui/tool.h:26`. Documents the convention for adding a new tool module: one `routes/<tool>.c/.h` pair exposing a single register function, called from `server.c`, routes prefixed `/api/<tool_name>/...`.

### Service layer (internal API between routes and Ursatz — not externally callable, but the seam the whole architecture is built around)
- `bool ursatz_gui_persistence_check(const char *db_path, char **out_message)` — `persistence_service.h:19`.
- `UrsatzGuiAnalysisResult ursatz_gui_analysis_import_and_save(const char *pieces_dir, const unsigned char *midi_data, size_t midi_size, char **out_id)` — `analysis_service.h:73`.
- `UrsatzGuiAnalysisResult ursatz_gui_analysis_list(const char *pieces_dir, char ***out_ids, size_t *out_count)` — `analysis_service.h:83`.
- `UrsatzGuiAnalysisResult ursatz_gui_analysis_get_piece(const char *pieces_dir, const char *id, UrsatzGuiPieceDetail **out_detail)` — `analysis_service.h:96`.
- `void ursatz_gui_piece_detail_destroy(UrsatzGuiPieceDetail *detail)` — `analysis_service.h:98`.

No CLI commands beyond the server binary itself; no client-callable JS functions (see §6 — the frontend doesn't call the analyzer API).

## 3. Internal architecture

```
main.c
  └─> server.c  (ursatz_gui_server_start/stop; wires routes, serves frontend/public/ as static root)
        ├─> routes/health.c   → service/persistence_service.c → Ursatz persistence_open/close
        └─> routes/analyzer.c → service/analysis_service.c
                                    → Ursatz: midi_importer, key_estimation, chord_identification,
                                      persistence_note_events/interpretation, common_practice_minimal
                                    → sqlite3 directly, for a ursatz-gui-owned index table
```

- Route files never call Ursatz or sqlite3 directly — a convention enforced by code review/comments, not the compiler.
- `analysis_service.c` maintains its own SQLite table (`ursatz_gui_interpretations`: id, interpretation_type) inside the same per-piece `.db` file Ursatz manages, opened via a second raw `sqlite3_open` connection. This works around Ursatz's persistence layer having no "piece" concept and no way to enumerate stored interpretations (only load-by-known-id) — a deliberate, heavily-commented workaround.
- Frontend (`frontend/public/`) is static and framework-free: `app.js` renders a hardcoded tool list in the sidebar and calls `/api/health/status` on load. `index.html`/`style.css` provide the shell chrome only.

## 4. Dependencies

**This repo depends on:**
- `Ursatz` (`Mitchell-Opitz/ursatz`) — vendored as a git submodule at `external/ursatz`, pinned to commit `e48a35f166460c1e51f0f5457f4fa240f50283fd`. Provides the core analysis/persistence engine (MIDI import, key estimation, chord identification, interpretation persistence) and the `common_practice_minimal` analysis framework, both linked into the backend.
- `civetweb` v1.16 — fetched via CMake `FetchContent` from the upstream civetweb GitHub repo, built with SSL/testing/C++/ASan all disabled. Provides the embedded HTTP server.
- `sqlite3` — used directly in `analysis_service.c` (pulled in transitively through the `ursatz` link target, not declared as its own explicit dependency).
- No JS/Node dependencies — no `package.json`, `node_modules`, or bundler; the frontend is hand-written static HTML/CSS/JS.

**What depends on this repo:** nothing evident in-repo. README and a code comment reference `ursatz-analyzer` as a sibling repo sharing the "same cross-repo pattern" — a design reference, not a build/runtime dependency in either direction. The `external/ursatz` submodule directory was not checked out in this working tree (only the gitlink/commit pin was present), so Ursatz's own headers were not read directly — module names used inside Ursatz (e.g. `core/`, `io/midi_importer.h`, `analysis/key_estimation.h`, `persistence/`, `pitch/`, `structure/`) are inferred from `#include` lines in `analysis_service.c`, not confirmed against the actual header files.

## 5. Current implementation status

**Built and working:**
- Server bootstrap, signal handling, civetweb wiring, static frontend serving.
- Health-check endpoint and its persistence-layer round trip.
- Full Analyzer backend API: MIDI upload → import/analyze/save, list pieces, get piece detail with interpretation summaries, path-traversal-safe ID handling.
- Integration test for the health endpoint (real server, raw-socket HTTP request/response assertions).

**Stubbed / planned / incomplete:**
- **Frontend has no code path for the Analyzer API at all.** The backend fully implements upload/list/detail, but `app.js` never calls `/api/analyzer/pieces*` — only the health endpoint. The "Analysis" tool is marked active in the sidebar but the page renders no actual analysis UI.
- **Corpus tool**: listed in the sidebar (`active: false`) and named in the README as existing "now" alongside Analysis, but has no backend route file and no frontend behavior — sidebar entry is decorative only.
- **Generator tool**: explicitly deferred, README marks it out of scope; sidebar shows it as "coming soon."
- No tests exist for the Analyzer routes or `analysis_service.c` internals, though both are compiled into the test binary.
- No authentication, no native app packaging — both explicitly out of scope per the README.

## 6. Internal inconsistencies

- **README vs. reality on Corpus**: the README's architecture description implies Corpus is a present-tense tool module ("Analysis/Corpus now, Generator later"), but no `routes/corpus.*` exists and the frontend only has an inert, unclickable sidebar entry for it.
- **Backend/frontend capability gap**: the Analyzer's upload/list/detail API is fully implemented and reachable, but nothing in the UI calls it — the sidebar advertises an active "Analysis" tool with no corresponding UI, only a health-status panel.
- **Decorative sidebar**: `style.css` sets `cursor: default` on tool-list items and `app.js` attaches no click handlers, so the sidebar looks like navigation but isn't — consistent with the above gap, but worth flagging as dead UI surface.
- **Unseeded RNG for piece IDs**: `generate_piece_id` (`analysis_service.c:439`) combines `time(NULL)` with `rand()`, but no `srand()` call exists anywhere in the codebase. Every fresh process run starts `rand()` from the same default seed, so ID generation is less random across process restarts than the code likely intends — a latent uniqueness footgun with no visible collision handling against `persistence_open`.
- **Undocumented side effect on health check**: hitting `/api/health/status` creates a file (`ursatz_gui_health_check.db`) in the process's working directory as a side effect of the check; this isn't mentioned in the README's description of the endpoint.
- **No TODO/FIXME/stub markers anywhere in the codebase** — unimplemented scope (Corpus, Generator, re-analysis, auth) is documented in README prose rather than in code, which is consistent rather than contradictory, but means the gaps above aren't self-flagged in the source the way a TODO would be.

---
*Note: `external/ursatz` (the submodule) was not checked out in this working tree — only its pinned commit reference was visible. Everything said above about Ursatz's internal module structure is inferred from `#include` paths in `analysis_service.c`, not from reading Ursatz's own source.*
