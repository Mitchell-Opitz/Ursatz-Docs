# Ursatz-Composer — Status

**Last verified against repo state:** N/A — this repo does not exist yet.

## Purpose (planned)

The generative composition engine — Phase 3 of `Overview/Apps_Roadmap.md`. "Blend the
whimsical aspect of this with the darkness of that, match project requirements, apply my
style, generate the piece." Motif-up or form-down, either direction. Will consume the
`Ursatz` library the same way `Ursatz-Analyzer` and `Ursatz-GUI` do.

## Current implementation

None. No repository exists.

## Gating dependencies (must be true before this repo starts)

1. Framework v1 verification actually completes (see `Ursatz-Library/Known_Gaps.md` and
   `Overview/Master_Roadmap.md`'s Phase 2.5 section) — not yet done.
2. A real ranking/preference layer exists — `Constraint`/`Preference` are currently
   deliberate kernel stubs (see `Ursatz-Library/Registry/Registry_Theory_Analysis.md`).
3. `StatisticalModel`/`StyleModel` become real, not stubbed.
4. A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult) — analysis
   output is queryable-by-construction already, but nothing queries it yet.

## When this repo is created

1. Fill in this file's Purpose/Current Implementation sections for real.
2. Create `Known_Gaps.md` alongside it, using `Ursatz-Analyzer/Known_Gaps.md` or
   `Ursatz-GUI/Known_Gaps.md` as a template.
3. Add a row for it to `Overview/Master_Roadmap.md`'s cross-repo status summary table.
4. Add a dated entry to `Reconciliation_Log.md` recording the repo's creation and initial
   verified state.

Nothing else in this docs repo needs to change.
