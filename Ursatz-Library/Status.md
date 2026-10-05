# Ursatz Library — Status

**Last verified against repo state:** 2026-10-05, `main` at `090695c` (PR #97). See
`Reconciliation_Log.md` for how this was checked.

## Purpose

A general-purpose, theory-neutral computational music platform, written as a C11 static
library (`libursatz`). Facts about music (notes, timing, structure) are kept strictly
separate from *interpretations* of those facts (chord, key, form), which are only ever
produced under an explicit, named, swappable "Theory Framework," never assumed by the
kernel. It exists to be a substrate other tools build on, and it has no CLI or executable of its
own. See `Overview/Architecture_Principles.md` for the governing design laws and layer model.

## The one-paragraph picture

The kernel (L0 to L9) is broadly built. On top of it sit 17 Theory Frameworks, most of them a
translation of Open Music Theory into rule libraries (tonality, harmony, counterpoint, form,
meter, notation). Those rule libraries take already-labeled data (scale degrees, inversions,
soprano degree) and say whether rules hold. **The missing piece is perception**: turning
imported notes into that labeled data. Today the analysis pipeline still runs only on the
small `common-practice-minimal` framework, chord identification fails on most real
multi-voice textures, and MIDI import ignores meter. Closing that gap is the next work; see
`Overview/Master_Roadmap.md`.

## Current implementation, by layer

**L0 Identity, L1 Core:** implemented. EntityID, Version, VersionModel, SchemaVersion; Entity,
Value, Occurrence base types; the five equality notions (identity, value and structural
implemented; semantic and canonical as caller-supplied-function contracts); VoiceReference,
RelationshipReference, ProvenanceReference, FrameworkReference.

**L2 Time / Pitch:** implemented.
- Time: Duration, MusicalTime, TimeSpan, TimeInstant, TemporalAnchor (exact rationals);
  TimeSignature, NoteValue, Tuplet, MetronomeMarking, HistoricalPeriod (PRs #52 to #56).
  MeterMap and TempoMap not started.
- Pitch: PitchIdentity (opaque), PitchSpelling, PitchClass, TuningSystem, PitchRealization,
  PitchNumericValue (exact-rational numeric position under a TuningSystem), WrittenPitch,
  SoundingPitch; Clef, StaffPosition, ClefResolution, Accidental, KeySignature,
  GenericInterval (PRs #49 to #60). The notation values are accepted at L2 as theory-free
  values (decision 2026-10-05, `Dependency_Layer_Specification.md`).

**L3 Events:** implemented. Event, NoteEvent, RestEvent, ControlEvent, Tie, Slur, Articulation,
Dynamics, TempoMarking, StructuralMarker, Instrument, InstrumentCapability.

**L4 Containers:** implemented. Container, Voice (real interval-graph-coloring assignment for
single-channel MIDI, PRs #43/#46), Layer, Staff, Part, Measure (MeterMap reference is a bare
EntityID), Idea and Subphrase (PRs #95/#96), Phrase, Section, Movement, Work, Edition,
Arrangement, Revision, SourceVariant.

**L5 Structures:** Structure, Motif, MotifOccurrence, Cadence, Theme, ThemeOccurrence,
PhraseOccurrence, Figure, all plain shells (id + EntityID-reference list). Detection lives in
L8. Placement re-confirmed 2026-09-09; see `Dependency_Layer_Specification.md`.

**L6 Context / Semantics:** implemented.
- Context (composable via parent), but no analyzer builds one yet.
- Semantics: Interval, Sonority, SonorityBuilder (attack-based, fixed quarter-note grid, PR
  #45), Scale, Contour, RhythmicPattern; plus 32 vocabulary enums (`CadenceType`,
  `VoicePart`, `NonChordTone`, ...) added for the frameworks (PRs #65 to #97). Rule for those
  enums: `Dependency_Layer_Specification.md`.
- Relationship and PredicateDefinition, in production use with `precedes`,
  `transformed_from`, `resembles`.
- Interpretation family: Interpretation (sole output of every analyzer), Assertion,
  Observation, Hypothesis, Evidence, Uncertainty (3 of 8 kinds), Chord, Key, ScaleDegree,
  Texture (PR #64).

**L7 Theory:** Framework, Rule, RuleEvaluation, Constraint, Preference, Heuristic, Tendency,
StatisticalModel, StyleModel all exist as base types. `constraint_evaluate()` and
`preference_rank()` are deliberate stubs that always return a documented placeholder.
Tendency, StatisticalModel and StyleModel are shape only (no fitting).

**Framework extension modules (`frameworks/`, 17):**
- `common-practice-minimal`: 7 diatonic triads, 4 seventh qualities, major/natural-minor keys.
  **The only framework the analyzers use.** Triad/seventh matching is by subset (PR #48).
- 16 added in PRs #49 to #97: standard-notation, standard-markings, western-periods,
  standard-rhythm, standard-meter, standard-tonality, standard-texture, standard-voicing,
  standard-counterpoint, sixteenth-century-style, high-baroque-fugue, baroque-ground-bass,
  galant-schemas, scale-harmonization, standard-harmony, standard-form. Full catalog and
  dependency edges: `Registry/Registry_Theory_Analysis.md`.
- All 16 are **rule libraries over caller-supplied data**: each rule has a `*_rule_holds()`
  predicate and frameworks report hard/soft violation bitmasks. Only parts of
  standard-tonality and standard-texture build Interpretations. None is called by the
  analysis layer or by either app yet.
- The MIDI key-signature scanner (PR #47) gives callers an authoritative tonic/mode when a
  file declares one; it does not yet produce the `KeySignature` value type.

**L8 Analysis, Generation, Transformation:**
- **Transform:** 12 concrete transforms, all implemented and tested (the original 11 plus
  IntervallicManipulation, PR #95).
- **Analysis:** FeatureExtraction plus 14 analyzers: Interval Analysis, Scale Analysis, Chord
  Identification (triads + sevenths, subset match), Key Estimation, Voice Leading
  (packaging-only), Cadence Detection + Classification (PAC/IAC/HC/plagal/deceptive from root
  motion; PAC vs. IAC reported as "authentic (unclassified)" without soprano data), Motif
  Detection (exact repetition/transposition), Phrase Segmentation, Phrase Boundary Detection,
  Period, Sentence, Form Structure and Thematic Return Detection. Most take caller-computed
  inputs; only Chord ID and Key Estimation call a framework.
- **Generation:** real base types (Intent, ConstraintExtraction, ConflictDetection,
  GenerationPlan, Candidate/CandidateSet, CandidateEvaluator, GenerationRecord). `Generator`
  and `IntentCompiler` are headers only; nothing generates music.

**L9 I/O, Persistence, Corpus:**
- **I/O:** MIDIImporter (SMF 0/1, note-on/off only, tempo-independent whole-note-fraction
  units, per-track voice assignment). Does not read time-signature or tempo meta-events. MIDI
  key-signature scanner (PR #47). **MusicXmlExport** (`src/export/`, single part, synthetic
  4/4 meter, `xml:id` on notes). No MusicXML/MEI/ABC import.
- **Persistence:** SQLite storage for NoteEvent (incl. PitchSpelling), Uncertainty (3 kinds),
  Interpretation (opaque claim), Chord/Key wrappers; busy-timeout fix (PR #42). No
  Relationship, Structure, Entity or Provenance repository.
- **Corpus / corpus_stats:** create/list corpora, add/remove/list piece refs; one statistic
  (most common chord). The 5-piece `framework-v1-reference-set` is registered, but its test
  checks registration only.

## Build & test

CMake 3.20+, C11 compiler, SQLite3 (pkg-config) as the only third-party dependency. 220 kernel
tests run via root `ctest` and CI (single Ubuntu job), including one architecture-guard test
per module and valgrind runs. **The 248 framework tests do not run from the root build or
CI**; each framework adds its tests only when built standalone. Run separately on
2026-10-05, 247/248 pass; the failure is `architecture_common_practice_minimal`. See
`Known_Gaps.md`. Ursatz's own `docs/BUILDING.md` has the workaround.

```
cmake -S . -B build && cmake --build build
ctest --test-dir build -C Debug --output-on-failure
```

## What depends on this repo

`Ursatz-Analyzer` (CLI) and `Ursatz-GUI` (web app), both as a git submodule at
`external/ursatz`, both pinned at `8a0c9b0` (PR #47). Neither has PR #48 (subset chord
matching) or any framework from PRs #49 to #97.

## Open branch

`feat/phrase-expansion` (14 commits, standard-form phrase expansion and contraction) is not
merged. Decision 2026-10-05: hold it until the perception work starts, then merge or drop it
under the perception path rule (`Dependency_Layer_Specification.md`).
