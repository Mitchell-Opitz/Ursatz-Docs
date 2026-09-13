# Ursatz Library — Status

**Last verified against repo state:** 2026-09-13, commit `9fd5895` (PR #46). See
`Reconciliation_Log.md` for how this was checked.

## Purpose

A general-purpose, theory-neutral computational music platform, written as a C11 static
library (`libursatz`). Facts about music (notes, timing, structure) are kept strictly
separate from *interpretations* of those facts (chord, key, form), which are only ever
produced under an explicit, named, swappable "Theory Framework" — never assumed by the
kernel. It exists to be a substrate other tools build on; it has no CLI or executable of its
own. See `Overview/Architecture_Principles.md` for the governing design laws and layer model.

## Current implementation, by layer

**L0–L4 (Identity, Core, Time/Pitch, Events, Containers):** implemented for the Phase-1
slice (EntityID, Duration, MusicalTime, TimeSpan, PitchIdentity, PitchSpelling, NoteEvent,
Voice, and the three reference types). PitchSpelling is now persisted end-to-end (PR #38).
Voice assignment is real: greedy interval-graph coloring by timing, scoped per track for
multi-track single-channel MIDI (PRs #43, #46) — not a naive single-voice assumption.
`TuningSystem`/`PitchRealization` types exist (`src/pitch/`) but nothing consumes them yet —
see `Known_Gaps.md`.

**L5 (Structures):** Motif, MotifOccurrence, Cadence, Theme, ThemeOccurrence,
PhraseOccurrence, Figure all implemented as plain shells (id + EntityID-reference-list),
detection logic deliberately deferred to L8. This placement was questioned and re-confirmed
correct during the 2026-09-09 session — see `Dependency_Layer_Specification.md`.

**L6 (Context/Semantics):** Sonority is now built by a real `SonorityBuilder`
(`src/semantics/sonority_builder.*`, PR #45) — attack-based, fixed quarter-note beat grid
(a stated v1 default, not adaptive; see `docs/modules/semantics.md` in the Ursatz repo).
Contour and RhythmicPattern implemented. `Relationship` (full L6 edge type) is implemented
and in real production use, with three predicates actually in play: `precedes` (Period
antecedent→consequent, Form section order), `transformed_from` (Sentence
statement→repetition→continuation), `resembles` (Form thematic return, Theme-to-Theme).
`Interpretation` implemented as a thin wrapper — the sole output mechanism for every
analyzer. Context, Interval, Scale, ScaleDegree not started.

**L7 (Theory):** `frameworks/common-practice-minimal` identifies 7 diatonic triads
(degree-keyed) plus 4 seventh-chord qualities — dominant7, major7, minor7, half-diminished7
(root-relative interval-set matching, fully-diminished7 deliberately excluded — requires
harmonic minor, out of scope) — and major/natural-minor keys. `Constraint`/`Preference`
remain deliberate, documented stubs (`constraint_evaluate()`/`preference_rank()` do real
null-checking but always return a documented placeholder). `Rule`, `Heuristic`, `Tendency`,
`StatisticalModel` not started.

**L8 (Analysis, Generation, Transformation):**
- **Transform:** all 11 concrete transforms implemented and tested (Transposition,
  Inversion, Retrograde, RetrogradeInversion, Augmentation, Diminution, Fragmentation,
  Expansion, Contraction, Displacement, Ornamentation).
- **Analysis — all planned analyzers now implemented:** Chord Identification (incl.
  sevenths), Key Estimation, Voice Leading (packaging-only — see `Known_Gaps.md`), Cadence
  Detection + Classification (derives PAC/IAC/HC/Plagal/Deceptive from real chord
  root-motion; PAC vs. IAC needs soprano-resolution data that doesn't exist anywhere, so it
  reports "authentic (unclassified)" rather than guessing), Motif Detection
  (Contour+RhythmicPattern sequence matching, exact repetition/transposition only), Phrase
  Segmentation (pre-existing, untouched) + Phrase Boundary Detection (real is_boundary from
  Cadence primary signal + Contour/RhythmicPattern discontinuity secondary), Period Detection
  (first real `Relationship` use, `precedes`), Sentence Detection (chains two
  `transformed_from` Relationships), Form Structure Detection + Thematic Return Detection
  (structural PAC/IAC-only section boundaries; `resembles` for recurrence).
- **Generation:** interface contract only (`Generator`, `IntentCompiler`); no concrete
  implementation. Not started.

**L9 (I/O, Persistence, Corpus, Applications):**
- **I/O:** MIDIImporter (SMF 0/1, note-on/off only, tempo-independent whole-note-fraction
  Duration units as of PR #39). **MusicXmlExport now implemented**
  (`src/export/musicxml_export.*`, PRs #40–41) — single-part note/measure serialization with
  `xml:id` on exported notes. MusicXML/MEI/ABC *import*, other renderers: not started.
- **Persistence:** SQLite-backed storage for NoteEvent (incl. PitchSpelling), Uncertainty (3
  of 8 kinds), Interpretation (opaque), thin Chord/Key wrappers. SQLite busy-timeout fix
  applied (PR #42, avoids immediate `SQLITE_BUSY` on concurrent open). No
  `RelationshipRepository`, `StructureRepository`, `EntityRepository`, or
  `ProvenanceRepository` — see `Known_Gaps.md`.
  See `Overview/Master_Roadmap.md`'s Phase 2.5 section — CorpusRegression verifies
  registration only, not analysis output.

## Build & test

CMake 3.20+, C11 compiler, SQLite3 (pkg-config) as the only third-party dependency. Tests are
hand-rolled standalone C executables (150+ across all modules as of PR #46) run via `ctest`,
plus one architecture-guard test per module (regex-scans `#include` lines against an
allowlist, fails on violation). CI: single Ubuntu job, configure → build → `ctest`.

```
cmake -S . -B build && cmake --build build
ctest --test-dir build -C Debug --output-on-failure
```

## What depends on this repo

`Ursatz-Analyzer` (CLI) and `Ursatz-GUI` (web app), both as a git submodule at
`external/ursatz`. No bindings, no examples directory in active use.
