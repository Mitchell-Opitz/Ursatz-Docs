# Ursatz-Composer — Status

**Last verified against repo state:** 2026-10-05. This repo does not exist yet.

## Purpose (planned)

The generative composition engine, Phase 3 of `Overview/Apps_Roadmap.md`. "Blend the
whimsical aspect of this with the darkness of that, match project requirements, apply my
style, generate the piece." Motif-up or form-down, either direction. Will consume the
`Ursatz` library the same way `Ursatz-Analyzer` and `Ursatz-GUI` do.

## Current implementation

None. No repository exists.

## Gating dependencies

Revised 2026-10-05: rule-checked sketching (phases C1 to C3 of
`Roadmaps/Ursatz-Composer_Roadmap.md`) may start once Ursatz L5.1 (real constraint
evaluation), L5.4 (Generator interface) and L5.6 (MIDI renderer) merge. The items below gate
style learning and self-critique (C5, C8), not the repo's creation.

1. Framework v1 verification actually completes, which in practice needs the perception slice
   (`Overview/Master_Roadmap.md`, Phases 2.5 and 2.7). Not yet done.
2. A real ranking/preference layer exists. `Constraint`/`Preference` are currently
   deliberate kernel stubs (see `Ursatz-Library/Registry/Registry_Theory_Analysis.md`).
3. `StatisticalModel`/`StyleModel` become real. Both exist as shapes only (no fitting).
4. A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult). Analysis
   output is queryable-by-construction already, but nothing queries it yet.

## When this repo is created

1. Fill in this file's Purpose/Current Implementation sections for real.
2. Create `Known_Gaps.md` alongside it, using `Ursatz-Analyzer/Known_Gaps.md` or
   `Ursatz-GUI/Known_Gaps.md` as a template.
3. Add a row for it to `Overview/Master_Roadmap.md`'s cross-repo status summary table.
4. Add a dated entry to `Reconciliation_Log.md` recording the repo's creation and initial
   verified state.

Nothing else in this docs repo needs to change.
