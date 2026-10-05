# Ursatz Gap Report

**Audited:** 2026-10-05, Ursatz `main` at `090695c` (PR #97), against every file in
`Ursatz-Library/`, `Overview/` and `Reconciliation_Log.md`. Inventory of what's new:
`Ursatz_Inventory.md`.

Four buckets:

- **A. Matches the docs.** No action.
- **B. In code, not in docs.** Keep and document it, or cut it.
- **C. In docs, but missing or different in code.** Fix the doc (or the code).
- **D. Contradicts the docs.** Decide which side wins.

---

## Fix first (a code defect, whatever else is decided)

**Framework tests never run in CI.** Every `frameworks/*/CMakeLists.txt` ends with:

```
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    enable_testing()
    add_subdirectory(tests)
endif()
```

When built from the repo root (as CI does), that condition is false. So none of the 248
framework tests, including all 17 framework architecture guards, are compiled or run. Root
`ctest` reports 220 tests, all kernel. Run separately for this audit, the framework suites
pass 247/248. The failure is `architecture_common_practice_minimal` (3 violations: the guard's
allowlist omits the module's own seventh-chord headers). It has been silently failing since
the 2026-09-09 seventh-chord work. This pattern predates PR #49; the new frameworks copied it.

Fix: add the framework `tests/` directories from the root when testing is enabled, then
correct the `common-practice-minimal` allowlist.

---

## A. Matches the docs

| Item | Evidence |
|---|---|
| Theory lives in pluggable frameworks, not the kernel (Law 11, TDD "Extension mechanism") | All 16 frameworks are separate targets under `frameworks/`; `grep` finds no `src/` file including a framework header |
| Layer leakage prohibitions (Dependency spec) | All kernel architecture-guard tests pass |
| Framework Output Contract shape | Each framework mirrors `common-practice-minimal`: a `Framework` singleton plus `Rule`/`Constraint` tokens |
| Exact symbolic time (Law 19) | No `float`/`double` introduced in any new code |
| "One scoped branch per task" (Master Roadmap, process rule 2) | 49 PRs, each one feature or one framework slice |
| `Constraint`/`Preference` remain deliberate stubs | `constraint_evaluate()` / `preference_rank()` still return their `*_NOT_IMPLEMENTED` placeholders; frameworks use their own `*_rule_holds()` instead of calling them |
| Known gap: MIDI import ignores time signature | Still true. A `TimeSignature` type now exists (PR #54), but `midi_importer.c` still never reads `0xFF 0x58` |
| Known gap: `matches_collection` one-directional containment | Still present, unchanged (`frameworks/common-practice-minimal/key_identification.c:38`) |
| Known gap: Problem B (chord matching on multi-voice textures) | Still open. `NonChordTone`/`BeatStrength` now exist, but `common-practice-minimal` chord identification doesn't use them |
| Known gap: Phase 2.5 corpus verification incomplete | Still open. None of the 49 PRs touches the analysis pipeline or the corpus |
| Known gap: no `RelationshipRepository` etc. | Still none |
| `docs/ARCHITECTURE.md` open issues | Unchanged |

---

## B. In code, not in docs

Every row is "keep and document, or cut." Default recommendation in the last column.

| # | What | Where | Recommendation |
|---|---|---|---|
| B1 | 16 new frameworks, ~82k lines | `frameworks/*` (see inventory) | Keep. Document in `Status.md` L7, Registry "Theory Frameworks", Technical Design phase table |
| B2 | 48 new kernel headers across L2, L3, L4, L6 and L8 | `src/` (see inventory) | Depends on decisions D2 and D3 |
| B3 | 6 framework-to-framework dependency edges | `baroque-ground-bass`, `galant-schemas`, `high-baroque-fugue`, `sixteenth-century-style`, `scale-harmonization` depend on `standard-counterpoint`; `scale-harmonization` also on `galant-schemas` | See D4 |
| B4 | A 12th concrete transform, `IntervallicManipulation` | `src/transform/intervallic_manipulation.*` (PR #95) | Keep. Docs say "all 11 concrete transforms"; update to 12 |
| B5 | `Idea` and `Subphrase` L4 containers | `src/containers/` (PRs #95, #96) | Keep. Add to Registry Containers and to the L4 row in the Dependency spec |
| B6 | `Texture` L6 interpretation type | `src/interpretation/texture.*` (PR #64) | Keep. Add to Registry Semantic Layer |
| B7 | 4 kernel types with no non-test consumer: `SchemaSense`, `SubjectPart`, `SubjectCharacter`, `TempoMarking` | `src/semantics/`, `src/events/` | Cut, or move to the framework that names them. They fail kernel admission criterion (2), "required by multiple subsystems" |
| B8 | Unmerged branch, 14 commits | `origin/feat/phrase-expansion` | Decide: merge, finish or drop |
| B9 | Test count grew from "150+" to 220 kernel tests plus 248 framework tests | `Status.md` "Build & test" | Update the number |

---

## C. In docs, but missing or different in code

### C1. Caused by or related to the new work

| Doc claim | Where | Reality |
|---|---|---|
| Phase 10 Species Counterpoint: "Not started"; Registry row "First–Fifth Species Counterpoint: Not Started" | `Technical_Design_Document.md` phase table; `Registry_Theory_Analysis.md` | Built: `standard-counterpoint`, species 1 to 5, 48 test files |
| Phase 11 Common-Practice Harmony: "Not Started ... do not assume it exists" | Same two files | Partly built as `standard-harmony`, but in a different shape. The Registry row describes a harmonic **analyzer** (depends on Context and Interpretation, feeds Harmonic Analysis). What exists is a **rule checker** over caller-supplied chord data that emits no Interpretation |
| Phase 18 Advanced Frameworks: "Not started"; listed examples (Schenkerian, Jazz, etc.) are optional | Same two files | Not those examples, but the same tier: `sixteenth-century-style`, `high-baroque-fugue`, `baroque-ground-bass`, `galant-schemas`, `scale-harmonization` are style-specific frameworks |
| Phase 2 Temporal/Score (Meter, Clef, Tempo, NotationPosition): "Not started" | Technical Design phase table | Partly built, but not as a score model: `TimeSignature`, `Clef`, `StaffPosition`, `MetronomeMarking`, `TempoMarking` are value types; there is no MeterMap/TempoMap |
| L7: "`frameworks/common-practice-minimal` ... is the real minimal instance" | `Status.md`, Registry `Framework` row | 17 frameworks now exist |
| "Transform: all 11 concrete transforms" | `Status.md` L8 | 12 (see B4) |

### C2. Already wrong before PR #49 (found while auditing)

The docs were stale-pessimistic before any of the new work. Each type below exists at PR #48
(`310ea46`) as a real, non-stub base type with tests. Most were added 2026-09-04 to 09-06. Yet
the docs mark each one "Not Started." This is the same failure mode the 2026-09-14 log entry
found for the Generation layer, at much larger scale.

| Type(s) | Lines at #48 | Docs say |
|---|---|---|
| `Context` (`src/context/`) | 65 | Not started (`Status.md` L6, Dependency spec L6 row, Technical Design phase 4) |
| `Interval` (`src/semantics/`) | 86 | Not Started (Registry Semantic Layer) |
| `Scale` (`src/semantics/`) | 68 | Not Started |
| `ScaleDegree` (`src/interpretation/`) | 66 | Not Started |
| `Evidence`, `Observation`, `Hypothesis`, `Assertion` (`src/interpretation/`) | 148 / 142 / 178 / 73 | Not started (Registry `Evidence`; Technical Design phase 4) |
| `Rule`, `RuleEvaluation` (`src/rules/`) | 57 / 64 | Not Started (Registry; `Status.md` L7) |
| `Heuristic`, `Tendency`, `StatisticalModel`, `StyleModel` (`src/theory/`) | 59 / 85 / 131 / 110 | Not Started / "no StatisticalModel/StyleModel exists yet" (`Status.md`, `Apps_Roadmap.md`, `Ursatz-Composer/Status.md`) |
| `Entity`, `Value`, `Occurrence` (`src/core/`) | 99 / 76 / 44 | "base categories not started" (Dependency spec L1 row) |
| `RestEvent`, `ControlEvent`, `Tie`, `Slur`, `Articulation`, `Dynamics`, `Instrument` (`src/events/`) | 44 to 101 each | "RestEvent/ControlEvent/attachments not started" (Dependency spec L3 row) |
| `Staff`, `Part`, `Measure`, `Layer`, `Movement`, `Work`, `Edition`, `Arrangement`, `Revision`, `SourceVariant` (`src/containers/`) | 51 to 113 each | "rest not started" (Dependency spec L4 row) |
| `WrittenPitch`, `SoundingPitch`, `PitchNumericValue` (`src/pitch/`) | 41 / 41 / 191 | Not mentioned |
| `FeatureExtraction`, `Interval Analysis`, `Scale Analysis` (`src/analysis/`) | 101 / 209 / 162 | Not Started (Registry Analysis) |

Related claims that are now wrong:

- **"`TuningSystem`/`PitchRealization` ... used by nothing"** (`Master_Roadmap.md`,
  `Status.md`, `Known_Gaps.md`). `PitchNumericValue`, `SoundingPitch`, `Scale`, and
  `midi_importer.c` all reference `TuningSystem`. The **functional** claim still holds: Voice
  Leading, Cadence Classification and Motif Detection don't compute real pitch values. Reword
  it to say that, not "used by nothing."
- **"Architecture-guard tests ... implemented and passing"** for every module (Dependency spec,
  `Status.md`). True for the kernel. False for frameworks (see "Fix first").
- **Commit hashes recorded as `<pending>`** in `Status.md` and `Known_Gaps.md` headers. PR #47
  is `8a0c9b0`, PR #48 is `310ea46`.

Before marking any of the above "Implemented," check each one for the shell vs. real-logic
distinction this repo already uses for L5 and Generation. Most look like real base types with
validation, not orchestration.

---

## D. Contradicts the docs (decide which side wins)

### D1. Registry-first rule was skipped entirely

- **Docs:** "Nothing is implemented in Ursatz without first being recorded in the System
  Registry" (`Master_Roadmap.md` process rule 1). "Undocumented inter-module dependencies
  rejected: any edge not in the Registry fails CI" (Dependency spec, architecture test map).
- **Code:** none of the 16 frameworks, 48 kernel headers or 6 framework edges appears in the
  Registry. The "fails CI" enforcement doesn't exist in CI (each guard is a hand-kept
  allowlist, and framework guards don't run).
- **Options:** (a) back-fill the Registry and keep the rule; (b) relax the rule to
  "Registry-first for kernel, Registry-after for frameworks."
- **Recommendation:** (a). The rule is cheap to satisfy after the fact here, and it's the only
  place cross-framework edges get reviewed.

### D2. Theory vocabulary in the kernel

- **Docs:** a new concept enters the kernel only if it is "(1) framework-neutral, (2) required
  by multiple subsystems, (3) not reasonably representable as an extension, (4) semantically
  stable, (5) introduces no theory leakage" (`Technical_Design_Document.md`, Extension
  mechanism). Law 11: the kernel doesn't depend on any particular theory. Law 25: the
  foundation remains small.
- **Code:** 32 enums in `src/semantics/` name common-practice and style-period concepts
  (`CadenceType`, `HarmonicFunction`, `PhraseModelLabel`, `SixFourType`, `AppliedChordKind`,
  `FugueRole`, `SchemaFunction`, `OstinatoType`, ...). Each says it "carries no framework
  claim." But the vocabulary itself is theory-specific. 24 of them are used by exactly one
  framework (fails criterion 2), and 3 by none (B7). They are representable inside the
  framework that uses them (fails criterion 3).
- **Options:** (a) move each single-consumer enum into its framework, keeping only
  multi-framework ones (`VoicePart`, `BeatStrength`, `NonChordTone`, `ImitationAnswer`,
  `CadenceType`) in the kernel or a shared "vocabulary" layer; (b) amend the admission rule to
  allow name-only vocabulary enums, and record why.
- **Recommendation:** (a) is the stricter reading of the existing docs. (b) is cheaper. This is
  a genuine architecture call for the project owner.

### D3. Notation concepts placed in L2

- **Docs:** notation is `music-notation`, an L9 module, "not started at the kernel level"
  (Dependency spec). Law 3: notation is not musical reality. Prohibition: "Time MUST NOT import
  Notation."
- **Code:** `Clef`, `ClefResolution`, `StaffPosition`, `Accidental`, `KeySignature` in
  `src/pitch/`; `NoteValue`, `Tuplet`, `TimeSignature`, `MetronomeMarking` in `src/time/`. They
  don't *import* a notation module (guards pass), but they *are* notation concepts living in L2.
  `HistoricalPeriod` (a style-history concept) is also in `src/time/`.
- **Options:** (a) relocate to a new `music-notation` module; (b) declare these as L2 value types
  (some, like `TimeSignature`, are arguably meter, not notation) and update the layer docs.
- **Recommendation:** split it. `TimeSignature`, `Tuplet`, `KeySignature` and `MetronomeMarking`
  are reasonable L2 values (the MIDI importer will need `TimeSignature` to fix the non-4/4 gap).
  `Clef`, `StaffPosition`, `ClefResolution` and `NoteValue` are notation. `HistoricalPeriod`
  belongs in `western-periods`.

### D4. Frameworks are documented as independent

- **Docs:** Registry heading "Theory Frameworks (independent modules)"; each advanced framework
  is "independently pluggable."
- **Code:** 5 frameworks link `standard_counterpoint`, and `scale-harmonization` also links
  `galant_schemas`. Their guards allowlist those edges, so this was deliberate.
- **Recommendation:** accept the edges (reusing `standard-counterpoint`'s line and interval
  model is reasonable), record them in the Registry, and reword "independent" to "independently
  selectable."

### D5. Duplicate concepts with no single owner

| Concept | Copies |
|---|---|
| Triad / seventh identification | `common-practice-minimal` (used by the analyzers and both apps) and `standard-tonality` (unused by any analyzer). In-repo docs say they "coexist" |
| Cadence vocabulary | `CadenceType` enum (PAC, IAC, half, subverted) vs. `cadence_classification.c` strings (PAC/IAC/HC/plagal/deceptive/"authentic (unclassified)") |
| Cadence-strength ordering | `period_detection.c` `cadence_rank()`, `standard-form`'s own order ("not standard-harmony's", per its doc), and `standard-harmony`'s. The Registry already calls the first "a stated default, not Registry-settled" |
| Key signature | `pitch/KeySignature` (PR #57) vs. `io/midi_key_signature` (PR #47), no link between them |
| Soprano degree for PAC vs. IAC | `standard-harmony` can classify PAC vs. IAC from a caller-supplied soprano degree; `cadence_classification.c` still reports "authentic (unclassified)" because no soprano data exists |

**Recommendation:** pick one owner per concept and record it in the Registry. The cadence one
matters most, since Period and Form detection already depend on it.

### D6. Work went somewhere other than the documented next step

- **Docs:** "Each phase is scoped to the minimum needed to unblock the next"
  (`Master_Roadmap.md`, `Apps_Roadmap.md`). Phase 2.5 corpus verification is "the actual
  remaining gap." The weighted chord tie-break is "now demonstrated ... real follow-on work."
- **Code:** all 49 PRs are framework vocabulary and rule checkers (Phases 10, 11 and 18 in the
  old numbering). None touches the analysis pipeline, the corpus, or chord identification. None
  reaches either app (both are pinned at PR #47).
- **Not a code problem.** But either the roadmap's priority still stands (and this work is
  banked groundwork), or the direction changed. Either way it needs a `Reconciliation_Log.md`
  entry, per that file's own rule on purpose and direction changes.

### D7. Frameworks don't use the claim pattern

- **Docs:** "A framework-relative claim ... is a generic `Interpretation` ... the default
  assumption for any future analyzer work" (`Architecture_Principles.md`).
- **Code:** apart from parts of `standard-tonality` and `standard-texture`, the new frameworks
  return `bool`s and violation bitmasks over plain structs. They emit no `Interpretation`,
  `Evidence` or `Relationship`.
- **Not strictly a violation**, since they are rule libraries, not analyzers. But turning any
  of them into analysis output will need an adapter that does follow the pattern. Worth one
  sentence in the docs so nobody assumes they already do.
