# Ursatz-Analyzer Gap Report

**Audited:** 2026-10-05, Ursatz-Analyzer `main` at `718c86a` (PR #5, merged 2026-09-13),
against `Ursatz-Analyzer/Status.md` and `Ursatz-Analyzer/Known_Gaps.md`.

**No code changes since the docs were written.** The findings below are doc fixes, plus one
stale in-repo README.

## A. Matches the docs

| Claim | Evidence |
|---|---|
| Fixed `char[32]` labels with silent truncation | `analysis_report.h:10-11`, `chord_pipeline.h:13`, `key_pipeline.h:13` |
| `analysis_report.c` is a 13-line file | 13 lines |
| Three-source key order: meta-event, then closing-chord-first fallback, then brute force | `key_pipeline.c` |
| Minimal-accidental flat spelling for fallback-derived key labels | `prefers_flat_spelling()`, `key_pipeline.c:110` and `:145` |
| `matches_collection` bug still reachable through brute force | Unchanged in the pinned Ursatz |

## B. In code, not in docs

Nothing new.

## C. In docs, but different in code

| Doc claim | Reality | Fix |
|---|---|---|
| Known_Gaps: "README.md contains only the title" | README now has a "Key estimation" section (commit `587dbb2`) | Remove the gap, or replace it with the next item |
| Status: "pinned commit `610e3c4...`" | Pin is `8a0c9b0` (Ursatz PR #47) | Correct the hash. Note that it is 50 Ursatz PRs behind `main` |
| Header: "commit `<pending>` (PR #5)" | PR #5 merge is `718c86a` | Fill in the hash |
| Status: "Only 4 PRs exist total" | 5 (PR #5 is described elsewhere in the same file) | Change 4 to 5 |

## D. Contradicts the docs (in-repo README vs. code)

The repo's own `README.md` disagrees with its code in two places. The docs repo's description
is the correct one.

| README says | Code does |
|---|---|
| Step 2 tries "the piece's **first and last** onset-grouped chord" | Tries the **closing** chord first, then the opening chord (the PR #5 mid-branch fix) |
| Flat spelling is used "**only** when the key signature ... has a negative count. Every other label, the brute-force scan ... stays on sharp-only spelling" | Fallback-derived labels also get flat spelling via `prefers_flat_spelling()` |

Fix: update the Analyzer README. This is a code-repo change, not a docs-repo change.
