# Ursatz-GUI — Known Gaps

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13), commit `877c52a` (PR #26).

## Confirmed still present

- **Unseeded RNG for piece IDs.** `generate_piece_id` (`analysis_service.c`) combines
  `time(NULL)` with `rand()`, but no `srand()` call exists anywhere in `backend/src`. Every
  fresh process run starts `rand()` from the same default seed — a latent ID-collision risk
  across process restarts, with no visible collision handling against `persistence_open`.
- **Accept/Dispute claim actions are present in the UI but disabled.** No promotion endpoint
  exists yet (`provenance_reference_create_present()` in Ursatz still has zero callers —
  every real interpretation still reports "absent").
- **Relationship persistence is an application-owned stopgap**
  (`ursatz_gui_relationships` table), not a real Ursatz-side `RelationshipRepository`. Retire
  it in favor of a real repository once/if one is built — see `Ursatz-Library/Known_Gaps.md`.
- **Generator tool is explicitly out of scope** — no route, no frontend affordance beyond
  whatever placeholder text exists. This is a disclosed, not-yet-planned gap, not a bug.
- **No authentication, no native app packaging** — both explicitly out of scope per this
  repo's own design.
- **MusicXML export uses a synthetic 4/4 meter grid** for all pieces, since Ursatz has no
  real meter data yet — cosmetic, not derived from the actual piece. Re-verified 2026-09-14:
  `analysis_service.c` sets a `ursatz:synthetic-meter` marker consumed by the frontend
  specifically to flag this as synthetic, not authored.

## Resolved since the previous documented state (closed gaps — do not re-flag these)

- ~~Frontend has zero code path to the Analyzer API~~ — **closed, PR #19.** The Analysis
  view now calls `/api/analyzer/pieces*` extensively and renders real claim data.
- ~~Corpus tool listed in sidebar but has no backend route or frontend behavior~~ —
  **closed, PRs #8, #18.** `routes/corpus.c` and the Collections grid UI are both real and
  wired.
**"Re-run analysis" without re-import is still not built** — re-verified 2026-09-14:
`app.js` renders a "Re-run analysis" button with `disabled: true`. Still an open item, not a
closed one — kept here rather than in the resolved list above.
