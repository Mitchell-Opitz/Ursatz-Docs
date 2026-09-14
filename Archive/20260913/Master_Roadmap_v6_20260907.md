# Master Roadmap v6 — Ursatz Ecosystem

Status as of 2026-09-09: Phase 1 complete, Phase 2 complete, Framework Stage (Phase 2.5's
theory-expansion work) code-complete but not yet verified against the fixed test corpus.
Supersedes v4 — reflects the full Framework-stage branch sequence (structures-minimal
through form-structure-detection) completed this session, corrects the L5/L6 structure
placement (reverted after an incorrect Registry decision was caught and fixed mid-session),
and records the Framework Output Contract decision that governs how all future analyzer
output is shaped.

---

## What this project is

**Ursatz** — a general-purpose, theory-neutral computational music platform, written in C. Facts about music (notes, timing, structure) are kept strictly separate from *interpretations* of those facts (chord, key), which are only ever produced under an explicit, named, swappable "Theory Framework" — never assumed by the kernel.

**Long-term goal:** analyze real music, then generate original music, encoding real music-theory knowledge as a rule/preference-based symbolic system — not ML.

**Repos:**
- `Ursatz` — the library (theory-neutral kernel + pluggable frameworks)
- `ursatz-analyzer` — CLI consumer app, depends on Ursatz
- `ursatz-gui` — GUI consumer app, in progress (Phase 2)
- (planned) a generation engine consumer app — Phase 3

---

## Process discipline (why this has worked — keep following it)

1. **Registry-first.** Nothing is implemented in Ursatz without first being recorded in the System Registry, maintained separately from the repos. Claude Code never sees architecture docs directly — the project owner is the checkpoint between architecture and implementation.
2. **One scoped branch per Claude Code task**, never a whole phase at once.
3. **Claude Code flags judgment calls before writing code**; architect reviews and approves first. This has caught real design issues early (e.g. the Theory Framework circular-dependency split, keeping `constraint_evaluate` a kernel stub, the L5-vs-L6 placement of Motif/Cadence/Theme/PhraseOccurrence, the phrase-segmentation naming collision, the Relationship-chain mechanism for Period/Sentence/Form).
4. **Root-cause fixes over symptom patches**, especially for real-world-data bugs (e.g. the key-detection exact-match bug was fixed by generalizing existing chord-level tolerance to key-level; the sentence-detection use-after-free was fixed by reordering, not patching around).
5. **Disclosed scope cuts, not silent gaps.** Every "not built yet" is documented at the point of the gap.
6. **Investigate before implementing, on every branch.** Every Framework-stage branch this session started with "confirm actual repo state before writing code" — and on the majority of them, the Registry's stated status was wrong. This is now a standing practice, not a one-off precaution (see Process note, bottom).
7. **Calibration reminder:** process quality and percent-of-vision-complete are different numbers. Process has been consistently disciplined; overall completion is still a small fraction of the long-term vision. Keep these separate.

---

## Where things stand

**Ursatz library — kernel:** L0–L8-Transform implemented and verified.

**Ursatz library — Theory/Analysis (Framework Stage, this session):**
- `frameworks/common-practice-minimal` now identifies 7 diatonic triads **plus** 4 seventh-chord qualities (dominant7, major7, minor7, half-diminished7 — fully-diminished7 deliberately excluded, requires harmonic minor which is out of scope), major/natural-minor keys only.
- **Voice Leading** — implemented, packaging-only. Detects/classifies nothing itself; packages caller-supplied motion/interval-perfection/bass-motion into Interpretation. Real computation from pitch data is blocked on TuningSystem/PitchRealization (kernel, not built) — see gap below.
- **Cadence Detection + Cadence Classification** — implemented. Classification derives PAC/IAC/HC/Plagal/Deceptive from real chord root-motion (scale-degree roots). PAC vs. IAC requires soprano-resolution data that doesn't exist anywhere yet; reports "authentic (unclassified)" rather than guessing.
- **Motif Detection** — implemented. Direct Contour+RhythmicPattern sequence matching over caller-supplied windows; exact repetition and transposition only (transposition-invariant by construction, since Contour is direction-only). Inversion/retrograde/fragmentation matching deferred.
- **Phrase Segmentation (pre-existing) + Phrase Boundary Detection (new)** — Phrase Segmentation already existed as a caller-supplied-classification packager (predates this session's other analyzers) and was left untouched. Phrase Boundary Detection computes real is_boundary from Cadence (primary) + Contour/RhythmicPattern discontinuity (secondary, default max_length_without_cadence = 8), delegates to Phrase Segmentation for the plain boundary Interpretation, and emits a second Interpretation recording which signal fired. Builds PhraseOccurrence via a transient Phrase (L4) id-minting pattern.
- **Period Detection** — implemented. First real use of Relationship (L6): antecedent/consequent linked via `precedes`, Interpretation targets the Relationship's id. Cadence-strength ordering (HC < deceptive < plagal < authentic-unclassified < authentic-IAC < authentic-PAC) is a stated default, not Registry-settled.
- **Sentence Detection** — implemented. Chains two Relationships (`transformed_from`, direction = derived-part→source) for statement→repetition→continuation. Continuation = Contour/RhythmicPattern prefix match, min 2 events, strictly shorter than statement. Fragmentation (music-transform) confirmed NOT reusable for detection (id-cloning only) — new comparison logic (SentenceContinuationMatching) was required.
- **Form Structure Detection + Thematic Return Detection** — implemented. Sections bounded only by structural cadences (PAC/IAC — the only types that reach tonic; unclassified authentic doesn't count). Thematic return uses `resembles` (not `transformed_from` — recurrence, not derivation), linking ThemeOccurrence ids directly, not Section ids.

**L5 Structure placement — corrected mid-session:** Motif, MotifOccurrence, Cadence, Theme, ThemeOccurrence, PhraseOccurrence, Figure all pre-existed this session as L5 shells (music-structure) — a plain id + EntityID-reference-list shape, detection logic deliberately deferred to L8. A Framework Output Contract decision incorrectly proposed relocating four of them to L6 as new Interpretation-shaped types; Claude Code caught the contradiction against real repo state before writing code, and the decision was corrected: **all seven stay at L5, unchanged.** The actual, correct pattern — confirmed working across every analyzer branch above — is: L5 shell (the thing) + a generic L6 Interpretation targeting it (the framework-relative claim about it), same relationship Chord already has to Sonority. No new named Interpretation-family types were created this session.

**Corpus:** `framework-v1-reference-set` registered via the existing Corpus API. 5 pieces: Bach (BWV Anh. 114), Clementi (Op. 36), Field (Nocturne No. 1), Field (Nocturne No. 5), Mozart (K.545) — all diatonic/public-domain, deliberately simpler than the original mixed game-music/classical list discussed, to keep Framework v1 finite. The 3 originally-discussed game pieces (Zanarkand, FF Main Theme, Aeris' Theme) are held as the natural v2 seed once chord vocabulary needs to grow past diatonic. `tests/fixtures/framework-v1-corpus/` contains the actual MIDI files; `CorpusRegression` test verifies registration only — **does not run analysis and check output.**

**Persistence:** SQLite-backed storage for NoteEvent, Uncertainty, Interpretation, and thin Chord/Key wrappers — unchanged this session.

**ursatz-analyzer (CLI):** Working end-to-end pipeline, verified against real commercial music (Pachelbel's Canon in D). Unchanged this session — none of the new analyzers have been wired into the CLI's real-MIDI pipeline yet (`pitch_class_util`-equivalent extension for real interval/motion computation also not done — see gap below).

**ursatz-gui:** PRs #10–#13 this session — piece-detail crash fixed (was heap-overflow in handle_get_piece's response buffer sizing, same root cause as the truncation bug below), provenance mislabeling fixed (frontend was inverting analyzed/hand-entered), original filename now captured/displayed alongside piece IDs, and all 7 Framework-stage analyzer types (Cadence, Motif, Phrase, Period, Sentence, Form, Thematic Return) wired into the upload pipeline and surfaced in the piece-detail view (previously only Key/Chord ran on upload). Relationship persistence has no home in Ursatz yet, so this used an ursatz-gui-owned SQLite table as a stopgap (see Dependency Spec). `external/ursatz` submodule bumped to pick up the newer analyzer modules this required.

**Known, disclosed gaps:**
- Chord-region matching's exact-match rigidity (pre-existing) — confirmed this session to extend to seventh-chord matching too, inherited by construction.
- **TuningSystem/PitchRealization absence is a concrete blocker**, not theoretical: Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif Detection's exact-window matching all only work on caller-supplied or Contour/RhythmicPattern-level data — none of them compute real values from raw MIDI pitch data. Nothing in the Framework-stage pipeline runs end-to-end on real pitch input yet without this.
- **Framework v1 verification is in progress, not complete.** Manual GUI pass started against `framework-v1-reference-set`; only Bach (BWV Anh. 114) checked so far, surfacing two open analysis-quality issues not yet triaged: (1) only a single chord Interpretation persists for the entire 430-note piece despite many onset groups, (2) thematic-return matching is producing dense near-total pairwise matches among phrase-boundary occurrences, suggesting a threshold/window bug rather than genuine recurring material. Clementi, Field Nocturne No. 1, Field Nocturne No. 5, and Mozart K.545 not yet checked (or re-checked, in Mozart's case).

---

## PHASE 2 — Bare-bones Library GUI (done)

- feat/persistence-minimal (Ursatz) — done.
- feat/gui-scaffold (ursatz-gui) — done.
- feat/corpus-minimal, feat/corpus-api (Ursatz) — done.
- feat/corpus-stats-minimal (Ursatz) — done.
- feat/corpus-tool (ursatz-gui) — done.
- feat/add-and-analyze-flow — done. "Re-run analysis" without re-import still not built.
- feat/provenance-view — partial, unchanged this session. `provenance_reference_create_present()` still has zero callers — every real interpretation still reports "absent."
- fix/analyzer-detail-response-truncation — known bug, scoped, still not started.

---

## PHASE 2.5 — Framework Stage (this session — code complete, verification pending)

Framework Output Contract decision (governs all analyzer output going forward): new
Interpretation-family concepts (Cadence, Motif, Theme, PhraseOccurrence, Period, Sentence,
Form/Section, Thematic Return) are never new named claim-types. They are either an existing
L5 shell + a generic L6 Interpretation targeting it, or — where a claim links two or more
structural elements — a Relationship (predicates used so far: `precedes` for sequential
order, `transformed_from` for derivation, `resembles` for recurrence) with an Interpretation
targeting the Relationship's id. This pattern is now proven across 4 branches (Period,
Sentence, Form, Thematic Return) and should be the default assumption for any future
Framework-stage work, not re-litigated per branch.

Branches, in dependency order:
- feat/framework-output-contract-decision — decision only, no code, Registry/spec updated directly.
- feat/framework-test-corpus-setup — done (PR #26). Fixture dir + Corpus registration, no MIDI-content assumptions.
- feat/structures-minimal — no-op (PR #27). Confirmed L5 shells already correct as committed.
- feat/semantics-minimal — no-op (PR #27... #s continue below). Confirmed Contour/RhythmicPattern already implemented and TDD-compliant.
- feat/chord-vocab-sevenths — done (PR #28). 4 seventh qualities, quality-keyed matching.
- feat/voice-leading-analysis — done (PR #29). Added bass_motion field + regression tests only; no new detection.
- feat/cadence-detection — done (PR #30). CadenceClassification added on top of pre-existing CadenceDetection.
- feat/motif-detection-minimal — done (PR #31). Added the actual Contour/RhythmicPattern matching step on top of pre-existing packaging.
- feat/phrase-segmentation — done (PR #32). New PhraseBoundaryDetection analyzer added alongside untouched pre-existing PhraseSegmentation.
- feat/period-detection — done (PR #33). First real Relationship use.
- feat/sentence-detection — done (PR #34). Relationship chaining extended to 3 parts; use-after-free found and fixed within the branch.
- feat/form-structure-detection — done (PR #35). Closes the bottom-up pipeline: harmony → motif → phrase → period/sentence → form.

**Still open under this phase:**
- Fix chord-region matching's exact-match rigidity (extends to sevenths now too).
- Run the actual verification pass against `framework-v1-reference-set` (see "Where things stand" above) — this is the phase's real exit criterion, not yet met.
- Extend `ursatz-analyzer`'s `pitch_class_util` (or build the kernel-side TuningSystem/PitchRealization) so Voice Leading and Motif Detection can compute real values from MIDI input instead of only packaging caller-supplied ones.

---

## PHASE 3 — Generative Composition Engine

Gated behind Phase 2.5's verification being real, not just its code existing. **The actual gap isn't more rules — it's a ranking/preference layer.** Registry already reserves the right concepts (`Preference`, `Heuristic`, `CandidateEvaluator`) — none built yet.

A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult, L9, music-query) also does not exist yet. This session's Framework Output Contract decision makes all analysis output structurally queryable by construction (real Relationship predicates, structured Interpretation claims), which should make standing up that query layer additive when the time comes — but it hasn't been built, and isn't needed until Phase 3 planning actually starts.

**Depends on, not yet built:** full L8-Generation, a more complete Theory Framework (chromatic harmony, secondary dominants — likely triggered by pulling the 3 game pieces into the corpus), real `StatisticalModel`/`StyleModel`.

Not broken down step-by-step yet — premature until Phase 2.5 verification is real.

---

## Summary table

| Phase | Repo | Status | Depends on |
|---|---|---|---|
| 0 | Ursatz | Done | — |
| 1 | ursatz-analyzer | Done | Phase 0 |
| 2 | Ursatz + ursatz-gui | Done | Phase 1 |
| 2.5 | Ursatz (Framework Stage) | Code complete, verification pending | Phase 2 |
| 3 | new repo (generation) | Not started | Phase 2.5 verified |

Each branch remains its own scoped Claude Code task, architect-review-then-implement.

---

## Process note (added this session)

Registry/Roadmap status lagged actual repo state on the majority of branches this session
(structures-minimal, semantics-minimal, voice-leading-analysis, cadence-detection,
phrase-segmentation all found real code already present or a Registry decision that
contradicted committed reality). Treat "Not Started" in these documents as unverified until
a branch's own investigation step confirms it — this is now standing practice, not a
one-off. Consider a periodic reconciliation pass (like the 2026-09-07 one) on a regular
cadence rather than only when a contradiction is stumbled into mid-branch.