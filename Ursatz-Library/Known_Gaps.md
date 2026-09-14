# Ursatz Library — Known Gaps

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13), commit `9fd5895` (PR #46).

Each entry: what's missing/limited, why it's a disclosed cut rather than a silent one, and
what it blocks.

## Concrete, currently-blocking

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

## Analyzer-specific limitations (by design, not bugs)

- **Chord-region matching exact-match rigidity.** This extends to seventh-chord matching too
  (inherited by construction); there is no tolerance for extra non-chord tones. Partially mitigated
  by the new beat-grid Sonority construction (see above) but not eliminated.
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
