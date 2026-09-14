# Master Roadmap — Ursatz Ecosystem

**Last verified against repo state:** 2026-09-13 (Ursatz @ PR #46, Ursatz-GUI @ PR #26,
Ursatz-Analyzer @ PR #4). See `Reconciliation_Log.md` for how this was checked.

## What this project is

**Ursatz** — a general-purpose, theory-neutral computational music platform, written in C.
Facts about music (notes, timing, structure) are kept strictly separate from
*interpretations* of those facts (chord, key), which are only ever produced under an
explicit, named, swappable "Theory Framework" — never assumed by the kernel.

**Long-term goal:** analyze real music, then generate original music, encoding real
music-theory knowledge as a rule/preference-based symbolic system — not ML.

**Repos:**
- `Ursatz` — the library (theory-neutral kernel + pluggable frameworks). Detail:
  `Ursatz-Library/Status.md`.
- `Ursatz-Analyzer` — CLI consumer app, depends on Ursatz. Detail: `Ursatz-Analyzer/Status.md`.
- `Ursatz-GUI` — GUI consumer app. Detail: `Ursatz-GUI/Status.md`.
- `Ursatz-Composer` (planned, Phase 3) — generation engine consumer app. No repo yet.

## Cross-repo status summary

| Repo | Purpose | Current state | Depends on |
|---|---|---|---|
| Ursatz | Theory-neutral kernel + pluggable theory frameworks | L0–L8 implemented; L9 (io/persistence/corpus) partially implemented; MusicXML export added | — |
| Ursatz-Analyzer | CLI: MIDI → Markdown key/chord report | Working end-to-end, low-churn, functionally frozen at key+chord (no Framework-stage analyzers wired in) | Ursatz |
| Ursatz-GUI | Browser UI: upload, analyze, browse, view score | Library/Collections/Analysis views all live; all 7 Framework-stage analyzers wired in; score rendering + claim highlighting working | Ursatz |
| Ursatz-Composer | Generative composition engine | Not started | Ursatz (Phase 3 gate: real Framework v1 verification, a ranking/preference layer) |

## Process discipline (why this has worked — keep following it)

1. **Registry-first.** Nothing is implemented in Ursatz without first being recorded in the
   System Registry (`Ursatz-Library/Registry/`), maintained separately from the repos. The
   project owner is the checkpoint between architecture and implementation.
2. **One scoped branch per task**, never a whole phase at once.
3. **Judgment calls get flagged before code is written**; architect reviews and approves
   first. This has caught real design issues early (the Theory Framework circular-dependency
   split, keeping `constraint_evaluate` a kernel stub, the L5-vs-L6 placement question below,
   the phrase-segmentation naming collision, the Relationship-chain mechanism for
   Period/Sentence/Form).
4. **Root-cause fixes over symptom patches** — e.g. the key-detection exact-match bug was
   fixed by generalizing existing chord-level tolerance to key-level, not patched around.
5. **Disclosed scope cuts, not silent gaps.** Every "not built yet" is documented at the
   point of the gap — see each repo's `Known_Gaps.md`.
6. **Investigate before implementing, AND before deciding.** A mid-session architecture
   decision on 2026-09-09 (relocating Motif/Cadence/Theme/PhraseOccurrence from L5 to L6) was
   made without checking real repo state, was wrong, and had to be reverted — see
   `Reconciliation_Log.md`. This is now standing practice, not a one-off.
7. **Documentation is verified, not assumed.** The same "check real state before trusting a
   status claim" discipline now applies to this docs repo itself — see
   `Reconciliation_Log.md`'s 2026-09-13 entry. **Treat any status in this repo as unverified
   until a dated log entry or a `Status.md` header confirms when it was last checked.**
8. **Calibration reminder:** process quality and percent-of-vision-complete are different
   numbers. Process has been consistently disciplined; overall completion is still a small
   fraction of the long-term vision.

## Phase overview

| Phase | Repo | Status |
|---|---|---|
| 0 | Ursatz | Done |
| 1 | Ursatz-Analyzer | Done |
| 2 | Ursatz + Ursatz-GUI | Done |
| 2.5 | Ursatz (Framework Stage: 7 new analyzers, seventh chords, real Relationship use) | Code complete. **Verification against the fixed 5-piece reference corpus is not complete** — see below. Two of the two known analysis-quality bugs surfaced during partial verification are now fixed in code (see `Ursatz-Library/Known_Gaps.md`), but the corpus hasn't been re-run to confirm, and 4 of 5 pieces (Clementi, Field No. 1, Field No. 5, Mozart) have never been checked at all. |
| 3 | Ursatz-Composer (new repo, not started) | Gated on Phase 2.5's verification actually completing, plus a ranking/preference layer that doesn't exist yet |

**Phase 2.5's real exit criterion, still not met:** run the `framework-v1-reference-set`
5-piece corpus through the full analysis pipeline and check the *output*, not just that the
pieces are registered. `CorpusRegression` (Ursatz) verifies registration only. A manual GUI
pass checked Bach only, surfaced two analysis-quality bugs (chord-Interpretation
non-uniqueness, thematic-return over-matching) — both since fixed in code (PRs #36, #37) but
**not re-verified against the corpus**, and the other 4 pieces were never checked at all.

**Why this waited on Ursatz-GUI's growth, rather than being neglected:** the original way to
inspect analyzer output was a flat text dump — every chord, every phrase, every cadence,
hundreds of lines per piece. That format made real verification practically impossible; no
one can reliably eyeball "is PAC actually happening at measure 12" out of a wall of text.
Ursatz-GUI's Library/Analysis/Score-tab work (PRs #17–#21) wasn't a separate feature track
competing with verification — it *was* building the verification instrument: real notation
rendering with claims highlighted on the actual notes they target is the only practical way
to confirm analysis output is correct, not just present. That tooling only became real with
PR #20/#21 (Score tab + note highlighting), which is why the full-corpus verification pass
is still pending now rather than having happened earlier — it wasn't deferred, it was
blocked on the thing that makes it checkable at all. Now that the tooling exists, running the
other 4 pieces through it is a live, actionable next step, not a stalled one.

## Known ecosystem-wide blocker

`TuningSystem`/`PitchRealization` (Ursatz kernel, `src/pitch/`) now exist as types but are
used by nothing — Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif
Detection all still only operate on caller-supplied or Contour/RhythmicPattern-level
(direction/duration-only) data, not real computed pitch values. This is a concrete blocker on
analysis quality for real MIDI input, not a theoretical one. See
`Ursatz-Library/Known_Gaps.md`.

## Governing principle for this document

Each phase is scoped to the minimum needed to unblock the next. Architecture detail
(Registry, layer specs) lives in `Ursatz-Library/`, not here. This document is cross-project
sequencing and current cross-repo state only — if a fact is specific to one repo, it belongs
in that repo's own `Status.md`, referenced from here, not duplicated here.
