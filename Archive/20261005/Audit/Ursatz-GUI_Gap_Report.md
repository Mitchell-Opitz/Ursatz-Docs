# Ursatz-GUI Gap Report

**Audited:** 2026-10-05, Ursatz-GUI `main` at `cf87ca3` (PR #27, merged 2026-09-13), against
`Ursatz-GUI/Status.md` and `Ursatz-GUI/Known_Gaps.md`.

**No code changes since the docs were written.** This repo was not part of the recent work. The
findings below are small doc fixes.

## A. Matches the docs

| Claim | Evidence |
|---|---|
| Unseeded RNG for piece IDs | `analysis_service.c:1685` uses `rand()`; zero `srand` calls in `backend/src` |
| "Re-run analysis" button disabled | `frontend/public/app.js:214`, `disabled: true` |
| Synthetic 4/4 meter marker | `analysis_service.c:2406` checks `ursatz:synthetic-meter` |
| Key estimation is a ported copy of Ursatz-Analyzer's logic | Still true; still inherits the `matches_collection` bug |
| Relationship persistence is an app-owned stopgap table | Unchanged |

## B. In code, not in docs

Nothing new.

## C. In docs, but different in code

| Doc claim | Reality | Fix |
|---|---|---|
| Header: "commit `<pending>` (PR #27)" | PR #27 merge is `cf87ca3` | Fill in the hash |
| Build & test: submodule pin "not yet re-verified against Ursatz's PR #47" | Pin is `8a0c9b0`, which **is** Ursatz PR #47 | State the pin plainly. Note that it is 50 Ursatz PRs behind `main` (#48 to #97) |

## D. Contradicts the docs

Nothing.

## Note for the decisions in `README.md`

The GUI does not get any of the new framework work until its submodule is bumped past PR #47,
and even then nothing will call it. That includes PR #48 (the chord subset-match stopgap).
Bumping is low risk: no kernel API used by the GUI changed in #48 to #97. The one behavior
change is PR #48's subset-match chord identification, which changes chord output. It is
optional until something in the GUI actually uses the new frameworks.
