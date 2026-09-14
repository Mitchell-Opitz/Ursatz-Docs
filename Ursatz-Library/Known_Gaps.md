# Ursatz Library — Known Gaps

**Last verified against repo state:** 2026-09-14, commit `<pending>` (PR #48, chord
subset-match stopgap).

Each entry: what's missing/limited, why it's a disclosed cut rather than a silent one, and
what it blocks.

## Concrete, currently-blocking

**MIDI import ignores the time-signature meta-event; `MusicalTime`'s denominator is
hardcoded to 4×PPQN (whole-note-relative).** `midi_importer.c` never reads a file's
`time_signature` meta-event, so any "measure index" computed from `MusicalTime` (e.g. the Bach
fixture test's `measure_index_of()`) only lines up with real notated measures in 4/4 pieces.
Surfaced 2026-09-14 while scoping a chord-identification subset-match test against Field 1
(`tests/fixtures/framework-v1-corpus/Field 1.mid`, which is 12/8, not 4/4) — a
measure-indexed regression test for that fixture, or any other non-4/4 corpus piece, cannot be
written correctly until this is fixed. Deliberately kept out of scope for the subset-match
branch; recommend a separate branch/ticket for meter-aware measure indexing before writing
measure-indexed tests against Field 1 (or Clementi/Mozart, if either turns out non-4/4 too).

**`TuningSystem`/`PitchRealization` exist but are unused.** Both types now exist in code
(`src/pitch/tuning_system.*`, `pitch_realization.*`, with their own tests), but a repo-wide
check found zero references to either from `voice_leading_analysis.c`,
`cadence_classification.c`, or `motif_detection.c`. The consequence is unchanged from before these
types existed: Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif
Detection all still only operate on caller-supplied or Contour/RhythmicPattern-level
(direction/duration-only) data, and none of them compute real values from raw MIDI pitch data.
This is a concrete, not theoretical, blocker on analysis quality for real MIDI input.

**Sonority construction is a fixed, non-adaptive grid.** `SonorityBuilder` (PR #45) replaced
naive exact-onset grouping with real attack-based beat-grid construction, a genuine fix, but
still a fixed quarter-note grid that doesn't vary by texture or tempo and doesn't infer
chord-change points. Extending this is new scope, not yet started.

**Framework v1 corpus verification is incomplete.** `framework-v1-reference-set` (5 pieces:
Bach BWV Anh. 114, Clementi Op. 36, Field Nocturne No. 1, Field Nocturne No. 5, Mozart K.545)
is registered, but `CorpusRegression` only checks registration, not analysis output. A manual
GUI pass checked Bach only and surfaced two bugs, both since fixed in code (see Resolved
below) but **not re-verified against the corpus**. Clementi, both Field pieces, and Mozart
have never been checked at all. This is Phase 2.5's actual, still-open exit criterion; see
`Overview/Master_Roadmap.md`.

**`RelationshipRepository` doesn't exist.** Ursatz-GUI needed to persist real `Relationship`
instances (Period/Sentence/Form/Thematic Return output) and couldn't wait, so it added its
own SQLite table as an application-layer stopgap (see `Dependency_Layer_Specification.md`).
Retire that table in favor of a real repository once/if one is built. The same applies to
`StructureRepository`, `EntityRepository`, `ProvenanceRepository`; none exist.

**`Constraint`/`Preference` are deliberate stubs.** `constraint_evaluate()` does real
null-checking but always returns a documented placeholder; `preference_rank()` is similarly
stubbed. This blocks any real Generation work (Phase 3/12).

**`docs/ARCHITECTURE.md`'s two "open issues" are still open, unchanged.** (1) A
Theory→Corpus forward-dependency framing: `Tendency`/`StatisticalModel`/`StyleModel` in
`theory/` are speced to eventually depend on Corpus (L9) concepts that don't exist yet, so
they currently accept corpus data only as opaque pre-computed values. This is expected given
the current phase, not a real discrepancy. (2) A "Reference-type Registry synchronization gap"
note describes `VoiceReference`/`RelationshipReference`/`ProvenanceReference` as "proposed
only, blocked," which is stale; those three were Registry-authorized and implemented well
before this pass. The in-repo doc should be corrected (a repo-side fix, not something this
docs repo can do directly).

## Resolved since the 2026-09-09 Framework Stage session (verify-worthy claims, now closed)

- **Chord-Interpretation non-uniqueness.** Previously, only a single chord Interpretation
  persisted for an entire 430-note piece. Fixed in PR #36: chord Interpretation EntityID is
  now derived from `sonority_target_id`, guaranteeing distinct Sonority occurrences yield
  distinct chord ids. Not yet re-verified against the reference corpus (see above).
- **Thematic-return over-matching.** Previously produced dense, near-total pairwise matches
  among phrase-boundary occurrences. Fixed in PR #37: a minimum-window-length floor is now
  required for a thematic-return match. Not yet re-verified against the reference corpus.

**`key_identification.c`'s `matches_collection` does one-directional pitch-class containment,
not set equality.** A chromatic/foreign tone doesn't disqualify a candidate tonic, which lets
wrong tonics pass. This is the final brute-force fallback used when no MIDI key-signature
meta-event is present and the calling analyzer's own chord-based fallback (see
`Ursatz-Analyzer/Status.md`) doesn't resolve it either; as of PR #47 (key-signature scanner)
and Ursatz-Analyzer PR #5 (chord-fallback tonic priority + flat spelling), it is rarely reached
for real files, but the bug itself is untouched, confirmed present, unfixed, on `main`. Ursatz-GUI's
own independently-maintained port of this same fallback logic (`analysis_service.c`, see
`Ursatz-GUI/Known_Gaps.md`) inherits it too, as of GUI PR #27. Flagged as out of scope for all
of these changes; recommend a dedicated branch if it needs closing.

## Analyzer-specific limitations (by design, not bugs)

- **Chord-region matching exact-match rigidity ("Problem B," `sonority_builder.h`) —
  subset-match stopgap shipped (PR #48), still fundamentally insufficient for real
  multi-voice textures.** `common_practice_identify_triad`/the seventh-chord matcher
  (`frameworks/common-practice-minimal/`) now accept a match when the triad/seventh's degrees
  are a *subset* of a beat's distinct degrees (extra degrees ignored as passing tones), naive
  scan-order tie-break, ambiguous multi-triad ties logged to stderr rather than resolved
  silently, no public API/signature changes. **Result against Field 1
  (`tests/fixtures/framework-v1-corpus/Field 1.mid`), confirmed 2026-09-14: only 61/394 beats
  resolved to any triad at all, and the matched sequence does not cleanly track the known
  I-V-I-I-I-V-I-I progression.** Root cause: subset-match only tolerates *one* non-chord degree
  per beat (3 chord tones + 1 passing tone = 4 distinct), but Field 1's melody+accompaniment
  texture routinely produces 2+ non-chord degrees per beat, which this stopgap does not handle
  at all — most of Field 1 still fails to match. This confirms weakness (2) from the original
  approval was not a hypothetical edge case but the dominant failure mode on real multi-voice
  input. Real disambiguation needs duration/beat-strength-weighted tie-breaking and tolerance
  for multiple simultaneous non-chord tones, both still deliberately deferred; this is now
  demonstrated, not speculative, follow-on work, not something to fold into further stopgap
  patching. The `test_framework_v1_corpus_subset_match_report` test added in PR #48 is
  informational only (only Field 1's key, Eb major, is independently verified; the other 4
  pieces run under a placeholder C-major tonic just to exercise the pipeline, so their printed
  chord labels are not meaningful) — it is not a pass/fail correctness gate and should not be
  read as one.
- **Cadence Classification's PAC vs. IAC.** This requires soprano-resolution data that doesn't
  exist anywhere in the codebase, so it reports "authentic (unclassified)" rather than guessing.
  This is correct, disclosed behavior, not a bug.
- **Motif Detection.** Exact repetition and transposition only (transposition-invariant by
  construction, since Contour is direction-only); inversion/retrograde/fragmentation matching
  is deferred to v2, to be pulled forward only if the fixed test corpus requires it.
- **Fragmentation transform is not reusable for detection/matching.** This was confirmed during
  Sentence Detection's development; it only clones caller-supplied event ids, with no comparison
  logic. New comparison logic (`SentenceContinuationMatching`) was built instead.

## Phase-0 spec debt (disclosed, tracked in `Technical_Design_Document.md`)

Several Phase-1 implementations (persistence, MIDI import, MusicXML export, Key
Estimation/Chord Identification) shipped ahead of their owning deliverable specs (Persistence,
I/O, Theory/Rule/Constraint) being formally written. See
`Technical_Design_Document.md`'s Phase-0 deliverable inventory for the full list of
unwritten specs.
