# Ursatz — Repository Analysis

## 1. Purpose

Ursatz is a general-purpose, theory-neutral computational music platform implemented as a C11 static library (`libursatz`). It provides a canonical intermediate representation of musical material — events, containers, structure, semantics, relationships, interpretations, and provenance — kept strictly separated by architectural layer, so that music-theoretic analysis, generation, and transformation logic can operate on the same underlying data model without hardcoding assumptions from any one theoretical tradition (e.g. common-practice tonal theory is implemented as a pluggable "framework," not baked into the core). It exists to be a substrate other tools (analyzers, generators, notation software, DAWs, corpus-study tools) could build on, though no such consumers exist in this repo yet — it is purely a library with no CLI or executable entry point.

## 2. Public API Surface

There is **no CLI and no `main()`** anywhere in the repo. `src/ursatz.c` is a 2-line library version stamp:
```c
int ursatz_version(void);  // returns 0
```
The "public API" is the collection of public headers under `src/**/*.h`, exposed via CMake's `target_include_directories(ursatz PUBLIC …)`. Everything below is a C-header interface (function + struct), not a runnable command.

**Analysis (`src/analysis/`)**
- `Analyzer` interface (`analyzer.h`): `AnalyzerFn analyze(input, context, config, out)` → `AnalysisResult` — generic contract every concrete analyzer implements; always emits `Interpretation`+`Evidence`, never a bare conclusion.
- `key_estimation_analyze()` (`key_estimation.h`) — deterministic key estimation (major/natural-minor match only) against a `Framework`; fails explicitly (`ANALYZER_ERR_ANALYSIS_FAILED`) rather than guessing.
- Similar `extern const Analyzer X_ANALYZER` exports for chord identification, interval analysis, scale analysis, voice-leading analysis, cadence detection, motif detection, phrase segmentation, feature extraction.
- Note: `chord_identification.c`/`key_estimation.c` are compiled into `frameworks/common-practice-minimal`, **not** into `libursatz`, despite headers living in `src/analysis/` (see §6).

**Generation (`src/generation/`)**
- `Generator` interface (`generator.h`): `GeneratorFn generate(plan, random_seed, configuration, out)` → `CandidateSet*` — rule/theory-driven candidate generation.
- `IntentCompiler` interface (`intent_compiler.h`): `IntentCompilerFn compile(request_text, context, config, out)` → `Intent*` — contract only; the NLP implementation is explicitly out of scope, no concrete implementation exists.

**I/O (`src/io/`)**
- `midi_importer_import(data, size, out_events, out_pitch_values, out_count)` (`midi_importer.h`) — parses SMF type 0/1 into `NoteEvent*` arrays. Built on `smf_reader_parse()` (header + track chunks) and `midi_event_decoder_decode()` (note-on/off only; skips CC/program-change/pitch-bend/meta/sysex; handles running status).

**Persistence (`src/persistence/`)**
- `persistence_open(path, out)` / `persistence_close(handle)` (`persistence.h`) — SQLite-backed, fail-fast. Explicitly documented as a minimal save/reload facility, not the full repository contract. Sub-APIs for note events, uncertainty, interpretation, chord, key persistence.

**Other module headers** (data-model primitives, not "callable APIs" in the traditional sense): `structure/` (motif, cadence, theme, figure + occurrences), `interpretation/` (key, chord, interpretation, evidence, uncertainty), `theory/framework.h`, `rules/` (constraint, preference, rule), `transform/` (11 concrete transforms: transposition, inversion, retrograde, retrograde-inversion, augmentation, diminution, fragmentation, expansion, contraction, displacement, ornamentation), `containers/` (voice, layer, staff, part, measure, phrase, section, movement, work, edition, arrangement, revision, source_variant), `events/` (note, rest, control events, articulation, dynamics, slur, tie, instrument).

Two build targets total: `ursatz` (the core static library) and `frameworks/common-practice-minimal` (a `Framework` implementation providing tonal key/chord/triad vocabulary, linked against `ursatz`).

## 3. Internal Architecture

The codebase enforces a strict 10-layer dependency model (documented in `docs/ARCHITECTURE.md`, verified via `#include` grep across all modules — no violations found):

```
L0 identity → L1 core → L2 time, pitch → L3 events → L4 containers → L5 structure
→ L6 context, semantics, relationship, interpretation → L7 theory, rules
→ L8 transform, analysis, generation → L9 io, persistence
```

Key relationships:
- `events` sits on `time`+`pitch`+`core`+`identity`; `containers` and `structure` reference material by id rather than embedding it (an occurrence/reference pattern).
- `analysis` (L8) is the heaviest consumer — pulls from `context`, `events`, `interpretation`, `semantics`, `structure`, `theory`: analyzers read the canonical IR plus a pluggable `Framework` and always emit `Interpretation`+`Evidence`.
- `generation` (L8) is rule/theory/context driven, never touching raw events directly.
- `transform` (L8) operates purely on `events`+`pitch`+`time`, independent of theory/interpretation — pure structural manipulation.
- `persistence`/`io` (L9) both depend on `events`/`pitch`/`time`; `persistence` additionally serializes `interpretation` (Chord/Key results). `io`'s MIDI importer produces only raw `NoteEvent`s, no analysis.
- `rules` legitimately imports `theory` (documented one-way sideways exception at L7); `theory` never imports back.

Architectural conformance is enforced by a custom per-module test (`tests/<module>/test_architecture_<module>.c`) that regex-scans each module's `#include "..."` lines against an allowlist and fails `ctest` on violation — this is a distinctive, deliberate design choice rather than convention alone.

## 4. Dependencies

**External:** CMake 3.20+, C11 compiler, and a single third-party library — **SQLite3** (via pkg-config), used only by `persistence/`. No other dependency manifest exists (no package.json/vcpkg.json/conanfile).

**Testing:** No third-party test framework (no cmocka/Unity/criterion). Tests are hand-rolled standalone C executables (138 test files across 18 modules, roughly one per source file) run via `ctest`, plus the 18 architecture-guard tests described above.

**CI:** `.github/workflows/ci.yml` — single Ubuntu job: configure → build → `ctest`. No lint/static-analysis step in CI despite `.clang-tidy`/`.clang-format` existing at the repo root.

**What depends on this repo:** Nothing in-repo. It's a standalone library with no CLI, no examples (`examples/` is empty, `.gitkeep` only), and no bindings. The only internal "consumer" is `frameworks/common-practice-minimal`, itself part of this repo.

## 5. Implementation Status

Overwhelmingly implemented, not stubbed. A grep for `TODO|FIXME|XXX|not implemented|stub` across `src/` and `frameworks/` returns only 10 hits, **all in header comments**, none in function bodies — and each references a specific "TDD" (Technical Design Document) section, suggesting deliberate, tracked scope cuts against a formal spec rather than abandoned work.

Confirmed real (intentional) stubs:
- `rules/constraint.c` — `constraint_evaluate()` does real null-checking but always returns `CONSTRAINT_EVAL_NOT_IMPLEMENTED` (documented as a placeholder per "TDD §96").
- `rules/preference.c` — `preference_rank()` is similarly a documented stub.
- `generation/constraint_extraction.h` / `conflict_detection.h` — declare the interface shape but not the algorithm ("out of scope for this pass").
- `persistence/persistence_uncertainty.h` — credible-interval / qualitative uncertainty persistence not implemented.
- `core/equality.h` — deferred, not implemented.

Sampling 18 `.c` files across every layer found substantive real logic (allocation, null-checks, business logic) everywhere else — no placeholder `return NULL`/`return 0` bodies outside the documented stubs above.

`docs/ARCHITECTURE.md`'s module-status table marks `pitch`, `events`, `containers`, `structure`, `semantics`, `interpretation`, `theory`, `rules`, `generation`, `io`, and `persistence` as "Implemented (partial)" — each has a corresponding `docs/modules/<name>.md` enumerating exactly what's deferred (e.g. `io` = MIDI import only; `persistence` = minimal SQLite storage for NoteEvent/Chord/Key only, not the full repository contract).

Test coverage is comprehensive across all 18 modules — not a gap area here.

## 6. Internal Inconsistencies

- **Stale README**: `README.md` states L9 (I/O adapters, notation, performance, corpus, applications) "has not started," but `io/` and `persistence/` (both L9) are actually partially implemented with real, working code (e.g. `midi_importer.c`, 349 lines). The README predates this work and wasn't updated.
- **`analysis/` build-target trap**: `chord_identification.c` and `key_estimation.c` live under `src/analysis/` with headers implying they build into `libursatz`, but they're actually compiled into `frameworks/common-practice-minimal` (to avoid a circular library dependency, since they depend on that framework, which depends on `ursatz`). Anyone linking only `libursatz` and expecting `KEY_ESTIMATION_ANALYZER`/`CHORD_IDENTIFICATION_ANALYZER` symbols will not find them — a documentation gap, not a code bug.
- **Two open issues tracked directly in `docs/ARCHITECTURE.md`**: (1) a "Theory → Corpus" forward-dependency contradiction — `Tendency`/`StatisticalModel`/`StyleModel` in `theory/` are speced to eventually depend on a Corpus (L9) concept that doesn't exist yet, so they currently accept corpus data only as opaque pre-computed values; (2) a "Reference-type Registry synchronization gap" — three reference types in `core/` (`VoiceReference`, `RelationshipReference`, `ProvenanceReference`) are implemented in code despite an external system-level Registry spec marking them "proposed only, blocked."
- No header/implementation signature mismatches were found on spot-check, and no dead/orphaned function declarations were found — the codebase is unusually self-consistent for its size, with the two caveats above being the only genuine discrepancies identified.

*Notes on method: architecture and dependency claims are derived from `#include` grep across `src/` and cross-checked against `docs/ARCHITECTURE.md`; implementation-status claims are based on a targeted grep for stub markers plus a sample of ~18 `.c` files rather than an exhaustive read of all ~200 files, so isolated undocumented stubs elsewhere are possible but unlikely given the consistency of what was sampled.*
