# Ursatz-GUI — Known Gaps

**Last verified against repo state:** 2026-09-14, commit `<pending>` (PR #27, key-detection
port).

## Confirmed still present

- **`analysis_service.c` maintains its own independent port of key-estimation logic
  instead of linking Ursatz-Analyzer as a library.** This is the second time the duplication
  has mattered in practice: Ursatz-Analyzer's key-detection fix (PR #5) didn't reach the GUI
  until ported separately here (PR #27), because `estimate_global_key` in `analysis_service.c`
  is a standalone copy, not a shared call. PR #27 kept the two in sync this time, but nothing
  enforces that going forward — any future Ursatz-Analyzer fix to key/chord/cadence logic needs
  a second, manual port to this file, or the GUI silently drifts again. Architect-level
  discussion (2026-09-14) concluded the real fix, having Ursatz-Analyzer's pipeline built as a
  linkable library and consumed here instead of vendored, requires a Registry-authorized new
  inter-module dependency, extracting Ursatz-Analyzer's pipeline out of its CLI-only structure,
  and untangling this file's persistence-interleaved analysis calls; deliberately deferred as
  its own larger, separately-tracked effort rather than folded into this scoped port.
  `key_identification.c`'s `matches_collection` containment bug (see
  `Ursatz-Library/Known_Gaps.md`) is inherited by this duplicate copy too.

- **Unseeded RNG for piece IDs.** `generate_piece_id` (`analysis_service.c`) combines
  `time(NULL)` with `rand()`, but no `srand()` call exists anywhere in `backend/src`. Every
  fresh process run starts `rand()` from the same default seed, a latent ID-collision risk
  across process restarts, with no visible collision handling against `persistence_open`.
- **Accept/Dispute claim actions are present in the UI but disabled.** No promotion endpoint
  exists yet (`provenance_reference_create_present()` in Ursatz still has zero callers, and
  every real interpretation still reports "absent").
- **Relationship persistence is an application-owned stopgap**
  (`ursatz_gui_relationships` table), not a real Ursatz-side `RelationshipRepository`. Retire
  it in favor of a real repository once/if one is built; see `Ursatz-Library/Known_Gaps.md`.
- **Generator tool is explicitly out of scope**, with no route and no frontend affordance
  beyond whatever placeholder text exists. This is a disclosed, not-yet-planned gap, not a bug.
- **No authentication, no native app packaging.** Both are explicitly out of scope per this
  repo's own design.
- **MusicXML export uses a synthetic 4/4 meter grid** for all pieces, since Ursatz has no
  real meter data yet. This is cosmetic, not derived from the actual piece. Re-verified
  2026-09-14: `analysis_service.c` sets a `ursatz:synthetic-meter` marker consumed by the
  frontend specifically to flag this as synthetic, not authored.

## Resolved since the previous documented state (closed gaps — do not re-flag these)

- ~~Frontend has zero code path to the Analyzer API~~ — **closed, PR #19.** The Analysis
  view now calls `/api/analyzer/pieces*` extensively and renders real claim data.
- ~~Corpus tool listed in sidebar but has no backend route or frontend behavior~~ —
  **closed, PRs #8, #18.** `routes/corpus.c` and the Collections grid UI are both real and
  wired.

**"Re-run analysis" without re-import is still not built.** Re-verified 2026-09-14:
`app.js` renders a "Re-run analysis" button with `disabled: true`. This is still an open
item, not a closed one, so it's kept here rather than in the resolved list above.
