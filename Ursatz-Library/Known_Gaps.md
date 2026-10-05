# Ursatz Library — Known Gaps

**Last verified against repo state:** 2026-10-05, `main` at `090695c` (PR #97).

Each entry: what's missing or limited, why it's a disclosed cut rather than a silent one, and
what it blocks. Ordered by what to fix first.

## 1. Fix first

**Framework tests don't run in CI.** Every `frameworks/*/CMakeLists.txt` adds its `tests/`
directory only when that framework is the top-level CMake project. From the root build (what
CI runs), all 248 framework tests, including 17 framework architecture guards, are skipped.
Run separately on 2026-10-05: 247/248 pass. Fix: add each framework's `tests/` from the root
when testing is enabled. This pattern predates PR #49 (`common-practice-minimal` had it).

**`architecture_common_practice_minimal` fails.** Its allowlist omits the module's own
`chord_vocabulary_seventh.h` and `seventh_identification.h` (3 violations). Silently failing
since the seventh-chord work (2026-09-09) because of the gap above. Fix the allowlist in the
same branch.

## 2. Blocking analysis quality (the perception gap)

The frameworks hold most of the theory the analyzer needs, but nothing turns imported notes
into their inputs. These four gaps are that bridge, in dependency order.

**MIDI import ignores time signatures.** `midi_importer.c` never reads `0xFF 0x58`;
`MusicalTime` is whole-note-relative, so any measure index only lines up with notated
measures in 4/4. Field Nocturne No. 1 is 12/8. `TimeSignature` (PR #54) now exists as the
value the importer should produce, and `standard-meter` can derive beat strength from it.
Blocks: meter-aware tests, beat strength, measure-indexed corpus checks.

**Chord identification fails on real multi-voice texture ("Problem B").** Subset matching
(PR #48) tolerates one non-chord degree per beat. Field 1 resolves only 61 of 394 beats to
any triad, and the matches don't track the known I-V-I progression, because
melody-plus-accompaniment routinely puts two or more non-chord tones in a beat. Needed:
duration and beat-strength weighting, using the `NonChordTone` and `BeatStrength` vocabulary
that now exists, then feeding results to `standard-tonality` Roman numerals.

**Sonority construction is a fixed quarter-note grid.** `SonorityBuilder` (PR #45) is
attack-based but doesn't adapt to meter, texture or tempo, and doesn't infer chord-change
points. Depends on the time-signature gap above.

**No analyzer computes melodic or soprano facts from pitch.** `PitchNumericValue` exists and
MIDIImporter returns one per note, but Voice Leading, Cadence Classification (PAC vs. IAC) and
Motif Detection still only take caller-supplied classifications. `standard-harmony` can tell
PAC from IAC given a soprano degree; nothing supplies it.

**Framework v1 corpus verification is incomplete.** The 5-piece `framework-v1-reference-set`
is registered, but `CorpusRegression` checks registration only. A manual GUI pass checked
Bach only (two bugs found, fixed in PRs #36/#37, not re-verified). Clementi, both Field
pieces and Mozart have never been checked. This is Phase 2.5's exit criterion.

## 3. Architecture debt

**Vocabulary enums awaiting relocation or removal.** Under the 2026-10-05 rule
(`Dependency_Layer_Specification.md`), 24 single-framework enums in `src/semantics/` move into
their framework the next time that framework changes. `SubjectPart`, `SubjectCharacter` and
`SchemaSense` have no consumer and should be removed.

**Duplicate concepts without a single owner.** Decided owners (2026-10-05), not yet migrated:
- *Cadence vocabulary:* `CadenceType` (L6 enum) is the owner. `cadence_classification.c`
  still emits strings ("authentic (PAC)", "authentic (unclassified)", plagal, deceptive);
  `CadenceType` lacks plagal and deceptive. Extend the enum and move the analyzer to it.
- *Cadence strength order:* three versions exist (`period_detection.c` `cadence_rank()`,
  `standard-form`, `standard-harmony`). `standard-harmony` is the owner; the other two should
  read from it.
- *Triad and seventh identification:* `common-practice-minimal` (used by analysis) and
  `standard-tonality` (unused by analysis). `standard-tonality` is the target owner;
  `common-practice-minimal` stays until the perception work moves Chord ID onto it.
- *Key signature:* `KeySignature` (L2) is the owner; `midi_key_signature_scan` should produce
  one.

**`key_identification.c`'s `matches_collection` uses one-directional containment.** A
candidate key matches when all seven of its pitch classes are present; foreign pitch classes
don't disqualify it, so wrong tonics pass. It is the last-resort fallback in Ursatz-Analyzer
and Ursatz-GUI (both have their own copies of the calling logic), rarely reached, still wrong
when reached.

**No Relationship, Structure, Entity or Provenance repository.** Ursatz-GUI persists
Relationships in its own stopgap table (`ursatz_gui_relationships`). Retire it once a
`RelationshipRepository` exists.

**`Constraint`/`Preference` evaluation is stubbed.** `constraint_evaluate()` and
`preference_rank()` validate arguments and return a placeholder. Frameworks check rules
through their own `*_rule_holds()` instead. Blocks real Generation (Phase 12).

**Theory to Corpus (open by design).** `Tendency`/`StatisticalModel`/`StyleModel` accept corpus
data only as opaque pre-computed values. Expected for the current phase.

**Unwritten specs.** Several implementations shipped ahead of their owning deliverable specs
(Temporal, Pitch, Container, Relationship, Context, Theory/Rule, Persistence, I/O). See
`Technical_Design_Document.md`'s Phase-0 inventory.

## 4. Analyzer-specific limits (by design)

- **Cadence Classification's PAC vs. IAC** reports "authentic (unclassified)" without soprano
  data. Correct, disclosed behavior.
- **Motif Detection** covers exact repetition and transposition only; inversion, retrograde
  and fragmentation matching are deferred.
- **Fragmentation transform is not reusable for detection.** It only clones caller-supplied
  event ids; Sentence Detection has its own comparison logic.
- **`test_framework_v1_corpus_subset_match_report`** (PR #48) is informational only: only Field
  1's key is independently verified, the other four run under a placeholder C-major tonic.

## Resolved (do not re-flag)

- Chord-Interpretation non-uniqueness: fixed PR #36.
- Thematic-return over-matching: fixed PR #37 (minimum window length).
- Chord matching required exact cardinality: relaxed to subset match, PR #48.
- `docs/ARCHITECTURE.md`'s stale "reference-type Registry synchronization" open issue: removed
  in the Ursatz repo on 2026-10-05 (branch `docs/reconcile-2026-10-05`).
- Docs listed ~35 implemented kernel types as "Not Started" (Context, Interval, Scale,
  ScaleDegree, Evidence, Rule, Heuristic, Tendency, StatisticalModel, StyleModel, containers,
  events, and others): corrected 2026-10-05.
