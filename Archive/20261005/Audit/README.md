# Audit: Code vs. Docs (2026-10-05)

**Archived.** The decisions below were made and applied the same day; see
`Reconciliation_Log.md`'s 2026-10-05 entry. This folder is the evidence behind that entry.

Read-only gap audit of all three code repos against this docs repo. Nothing in the code repos
or in the rest of this docs repo was changed. This folder covers steps 1 to 3 of the cleanup
plan (list what exists, compare to docs, sort into buckets). Step 4 (decide, then fix) is still
to do. Its open decisions are listed below.

## Repo state audited

| Repo | Docs last described | Actual head | Gap |
|---|---|---|---|
| Ursatz | PR #48 (`310ea46`) | PR #97 (`090695c`) | **49 merged PRs, 2026-10-02 to 2026-10-04, ~93k lines added, none documented** |
| Ursatz-GUI | PR #27 | PR #27 (`cf87ca3`) | none; only small doc errors |
| Ursatz-Analyzer | PR #5 | PR #5 (`718c86a`) | none; only small doc errors |

Both apps pin `external/ursatz` at `8a0c9b0` (Ursatz PR #47). None of the 49 new PRs reaches
either app.

## Files

| File | What it is |
|---|---|
| `Ursatz_Inventory.md` | Step 2: everything added in Ursatz PRs #49 to #97, by PR, framework and kernel layer |
| `Ursatz_Gap_Report.md` | Step 3 for the library: the four buckets, with evidence |
| `Ursatz-GUI_Gap_Report.md` | Step 3 for the GUI |
| `Ursatz-Analyzer_Gap_Report.md` | Step 3 for the CLI |

## Bottom line

The new work is in good shape **as code**. It builds clean with zero warnings, 220/220
top-level tests pass, and 247/248 framework tests pass (the one failure predates the new
work). It follows the existing Framework-extension shape, and the kernel never includes a
framework.

The problems are **process and architecture fit**, not quality:

1. **The 248 framework tests never run in CI.** Each framework only registers its tests when
   built standalone, so `ctest` at the root skips all of them. This is the one real defect to
   fix first, whatever else is decided.
2. **Nothing went through the Registry.** That breaks the project's #1 process rule.
3. **48 new kernel headers**, most of them common-practice theory vocabulary
   (`CadenceType`, `FugueRole`, `HarmonicFunction`, ...), went into `src/`. Most are used by
   exactly one framework. That is in tension with the kernel admission rule and Laws 11 and 25.
4. **The frameworks are rule checkers over caller-supplied data.** None of them is wired into
   the analysis pipeline or either app. They do not touch the Phase 2.5 verification gap,
   which is still the documented next step.
5. **Incidental finding:** the docs were already badly stale-pessimistic *before* this work.
   About 35 kernel types the docs call "Not Started" have existed since 2026-09-04 to 09-06.
   See bucket C of `Ursatz_Gap_Report.md`.

## Decisions needed (step 4)

Ordered by how much they block everything else.

1. **Kernel vocabulary enums.** Keep the 32 theory enums in `src/semantics/` (amend the
   kernel admission rule to allow framework-neutral *name-only* enums), or move each one into
   the framework that uses it? The latter is the stricter reading of the current docs.
2. **Notation types in L2.** `Clef`, `StaffPosition`, `Accidental`, `KeySignature` (pitch) and
   `NoteValue`, `Tuplet` (time) are notation concepts placed in the kernel. Keep them and
   update the layer docs, or relocate them to a `music-notation` module?
3. **Framework-to-framework dependencies.** Six edges exist (five frameworks depend on
   `standard-counterpoint`, `scale-harmonization` also depends on `galant-schemas`). Bless
   them in the Registry, or require the frameworks to be independent as documented?
4. **Phase mapping.** `standard-counterpoint` and `standard-harmony` look like Phases 10 and 11,
   and the fugue/galant/16th-century/ground-bass frameworks look like Phase 18. Count them as
   those phases (partial), or as a new "framework vocabulary" track? The Phase 11 deliverable
   was a harmonic *analyzer*. What got built is a rule checker.
5. **Unmerged branch** `feat/phrase-expansion` (14 commits ahead of main). Merge, finish or drop.
6. **Duplicate concepts.** Two triad identifiers (`common-practice-minimal`, `standard-tonality`)
   and three cadence-strength orderings (`period_detection.c`, `standard-form`,
   `standard-harmony`). Pick one owner for each, or document them as intentional.
7. **Next work.** The docs' stated next step (Phase 2.5 corpus verification, and the weighted
   chord tie-break that Field 1 proved necessary) is unchanged and still open. Confirm it is
   still the priority, or record the direction change in `Reconciliation_Log.md`.

Once these are decided: write the Registry rows, update `Status.md`/`Known_Gaps.md` for all
three repos, and add a `Reconciliation_Log.md` entry for this pass.
