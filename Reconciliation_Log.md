# Reconciliation Log

Append-only. Every entry is dated and never edited after the fact — if something in an old
entry turns out to be wrong, add a new entry correcting it rather than rewriting history.
This log records: (1) verification passes (docs checked against real repo state), and
(2) purpose/direction changes for any repo or the project as a whole.

---

## 2026-09-04 — Reconciliation pass (pre-restructure)

Read-only audit of Ursatz at commit `8768d8e`, verifying `docs/ARCHITECTURE.md`'s claims
layer by layer. Superseded by the 2026-09-07 pass below; original file archived
(`Archive/20260907/reconciliation_report.md`).

## 2026-09-07 — Reconciliation pass + condensing

Three per-repo self-analyses run (Ursatz, Ursatz-Analyzer, Ursatz-GUI), compared against
System Registry v3. Found the Registry's Status column lagging real repo state on six
entities (Framework, Constraint, Preference, Key Estimation, Chord Identification,
MIDIImporter — all marked "Not Started" despite real implementation). Corrected. All
project docs condensed from verbose originals (see `Archive/20260907/` for the pre-condense
versions). Files from this pass are now themselves superseded — see 2026-09-13 below —
and archived to `Archive/20260913/`.

## 2026-09-09 — Framework Stage session

Large development session across Ursatz and Ursatz-GUI: seven new analyzer types (Cadence
Detection/Classification, Motif Detection, Phrase Boundary Detection, Period Detection,
Sentence Detection, Form Structure Detection, Thematic Return Detection), seventh-chord
support, first real use of `Relationship` (predicates `precedes`, `transformed_from`,
`resembles`), a 5-piece fixed reference corpus, and (in Ursatz-GUI, PRs #10–#13) a
heap-overflow fix, a provenance-mislabeling fix, filename capture/display, and wiring all
seven new analyzer types into the upload/detail pipeline. Documented at the time in
Master Roadmap v6 / System Registry v7 / Dependency & Layer Specification v5 (now archived,
superseded by the 2026-09-13 pass below).

An architecture decision made mid-session (relocating Motif/Cadence/Theme/PhraseOccurrence
from L5 to L6) was caught as wrong later the same session, after checking actual committed
repo state, and reverted. Standing lesson recorded at the time: **verify actual repo state
before making an architecture decision, not just before implementing one.** This reconciliation
system exists partly because of that lesson — see the 2026-09-13 entry for how it's now
enforced as a recurring practice rather than a one-off.

## 2026-09-13 — Full reconciliation + documentation restructure

**What triggered this:** a user-requested full read of all docs + all three repos, to check
whether the docs matched real repo state. They didn't, in both directions — some doc claims
were stale-pessimistic (describing gaps already closed in code), and the docs as a whole were
stale-optimistic in scope (silently missing a large amount of real, shipped work newer than
any doc described).

**Findings, by repo:**

- **Ursatz** — confirmed all 2026-09-09 Framework Stage claims accurate (seventh chords,
  all 7 new analyzers, real `Relationship` usage, L5 shell placement, the 5-piece corpus).
  Found the docs' claim that `TuningSystem`/`PitchRealization` "don't exist yet" is **wrong**
  — both types now exist in code (`src/pitch/tuning_system.*`, `pitch_realization.*`, with
  tests) — but no analyzer actually consumes them yet, so the *functional* limitation the
  docs described (Voice Leading / Cadence Classification / Motif Detection can't compute real
  values from raw pitch) is still accurate; only the "doesn't exist" framing was wrong.
  Found real git history through **PR #46** (docs described only through #35/#46's
  predecessors) with substantial undocumented work since the Framework Stage session:
  chord-Interpretation EntityID uniqueness fix (PR #36, fixes the "only one chord persists
  for a 430-note piece" bug the docs called an open triage item), thematic-return matching
  minimum-window-length fix (PR #37, fixes the "dense near-total pairwise matches" bug the
  docs called an open triage item), PitchSpelling persistence (PR #38), tempo-independent
  MIDI Duration units (PR #39), a new MusicXML export module (`src/export/musicxml_export.*`,
  PRs #40–41), a SQLite busy-timeout fix (PR #42), real MIDI voice assignment via
  interval-graph coloring replacing a naive single-voice assumption (PR #43, extended to
  multi-track in PR #46), and beat-grid attack-based Sonority construction
  (`src/semantics/sonority_builder.*`, PR #45) — a real, if still fixed-grid/non-adaptive,
  fix for the previously-documented "chord-region exact-match rigidity" gap.
  `docs/ARCHITECTURE.md`'s two "open issues" are still open in-repo and unchanged.

- **Ursatz-GUI** — confirmed all PR #10–#13 claims accurate. Found real git history through
  **PR #26** (docs described only through #13) with a large amount of undocumented, shipped
  work: a Library view (rename/delete pieces, PR #17), a Collections grid replacing the
  inert Corpus sidebar entry (PR #18 — the docs' "Corpus is decorative only" gap is now
  **closed**), a tabbed Analysis view with claim detail/Evidence/Provenance panels (PR #19 —
  the docs' "frontend has zero code path to the Analyzer API" gap is now **closed**), a
  MusicXML export endpoint and OSMD-based Score tab (PR #20), and note-level highlighting
  tying score notes to selected claims (PR #21). The unseeded-RNG issue (`generate_piece_id`
  using `rand()` with no `srand()`) is still present and unfixed.

- **Ursatz-Analyzer** — confirmed genuinely low-churn: only 4 PRs total, the most recent
  (#4) a narrow submodule bump for a key-tolerance fix, not new functionality. All previously
  documented gaps (empty README, `char[32]` label truncation, single-region key estimation,
  exact-onset-only sonority grouping, dead-weight `analysis_report.c`) confirmed still present,
  unchanged.

**What changed as a result:** full restructure of this repo (see `README.md`) — per-repo
`Status.md`/`Known_Gaps.md` replacing point-in-time self-analyses, the Registry split by
layer, dated versions dropped from filenames in favor of "last verified" headers plus this
log. Old files moved to `Archive/20260913/`.

**Standing practice going forward:** the Framework Stage session already established
"verify before deciding" for architecture calls. This pass extends that to documentation
itself: **treat every status claim in this repo as unverified until a dated entry here (or a
`Status.md`'s own header) confirms when it was last checked against real repo state.**
A periodic reconciliation pass (this kind of check) should happen every time a repo's own
Roadmap-visible history advances meaningfully — not only when a contradiction is stumbled
into, and not left implicit as "someone should do this eventually."

## 2026-09-14 — Self-audit of the 2026-09-13 restructure

**What triggered this:** immediately after the 2026-09-13 restructure, a user-requested
self-check: re-verify the new docs against all three repos again. No repo had changed since
(`ursatz` still `9fd5895`/PR #46, `ursatz-gui` still `877c52a`/PR #26, `ursatz-analyzer`
still `38b72a9`/PR #4), so this was purely a check of whether the prior pass's writing was
itself accurate — the exact "verify, don't assume" discipline this project keeps re-learning
the hard way, applied to its own docs.

**Method:** three independent audit agents, one per repo, each given the relevant `Status.md`
+ `Known_Gaps.md` (plus a sample of Registry/spec claims for Ursatz) and told to verify every
falsifiable claim against real code/git history, skeptically.

**Findings:**

- **Ursatz-Analyzer** (`Status.md`, `Known_Gaps.md`) — all 9 checked claims CONFIRMED, zero
  discrepancies. Clean.
- **Ursatz-GUI** — all PR citations, view/route wiring, table names, RNG/Accept-Dispute/
  Relationship-stopgap claims CONFIRMED. Two items `Known_Gaps.md` had explicitly flagged as
  "not independently re-verified in the prior pass" (synthetic-meter marker, disabled
  re-run-analysis button) were checked and found **accurate** — the hedge was appropriately
  cautious but unnecessary; both now stated plainly, hedge removed.
  One real error found: **`Status.md` cited the wrong submodule pin hash** (`5d10667...`,
  which is actually a bump-*commit* hash in ursatz-gui's own history, not the submodule's
  pinned SHA). The real pin is `9fd5895` — Ursatz's PR #46 tip, i.e. fully current. Corrected.
- **Ursatz-Library** — the large majority of claims (PR #36–#46 fixes, TuningSystem/
  PitchRealization existing-but-unused, no repository classes, Uncertainty's 3-of-8 kinds,
  Constraint/Preference stubs) CONFIRMED with exact file/line citations. **One significant,
  repeated error found:** the Generation layer (`src/generation/`, ~1,159 lines) was
  described as "interface contract only, no concrete implementation, not started" in
  `Status.md`, and every one of `IntentCompiler`/`ConstraintExtraction`/`ConflictDetection`/
  `GenerationPlan`/`CandidateEvaluator`/`GenerationRecord` was marked "Not Started" in
  `Registry_Theory_Analysis.md` (with the same understatement echoed in
  `API_Contract_Specification.md` and `Test_Specification.md`). In fact, `Candidate`/
  `CandidateSet`, `CandidateEvaluator`, `ConflictDetection`, `ConstraintExtraction`,
  `GenerationPlan`, `GenerationRecord`, and `Intent` are all real, non-stub implementations
  with genuine validation logic — the same "base type implemented as a shell, no algorithm
  wired up yet" shape already correctly documented for L5's Motif/Theme types, just not
  recognized as the same pattern here. Only `Generator` and `IntentCompiler` (the top-level
  orchestration interfaces) are genuinely header-only. Corrected across all four files.

**Root cause of the Generation-layer miss:** whoever verified this section (in the prior
day's pass) apparently checked only the two named top-level interfaces
(`generator.h`/`intent_compiler.h`) rather than grepping the rest of `src/generation/` — an
easy trap when a directory's most prominent files are its unimplemented ones. Worth
remembering as a specific failure mode, not just "verify more": **when a module has both an
orchestration interface and supporting base types, check the base types separately — a
missing top-level interface doesn't mean the directory is empty.**

**Outcome:** three corrections made (one GUI submodule-pin hash, the Generation-layer status
across four Library files, two Known_Gaps hedges resolved to plain statements). Everything
else in the 2026-09-13 restructure held up under independent re-verification. Verification
headers on all audited files bumped to 2026-09-14.
