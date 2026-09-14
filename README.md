# Ursatz Project Documentation — Index

This repo is the single source of truth for the Ursatz ecosystem's design, status, and
history. It is maintained separately from the code repos; nothing here is generated from
CLAUDE.md files, and none of the code repos reference this repo back.

**Ecosystem:** `Ursatz` (theory-neutral C library / kernel), consumed by `Ursatz-Analyzer`
(CLI) and `Ursatz-GUI` (web app) today, with a planned `Ursatz-Composer` (generation engine)
later.

## Where to look

| I want to know... | Go to |
|---|---|
| The overall vision, current cross-repo status, what's next | `Overview/Master_Roadmap.md` |
| The product/business phase plan (not architecture) | `Overview/Apps_Roadmap.md` |
| The 25 non-negotiable design laws + layer model, at a glance | `Overview/Architecture_Principles.md` |
| Whether the *library* has built X | `Ursatz-Library/Status.md` |
| A known bug/limitation in the library | `Ursatz-Library/Known_Gaps.md` |
| The exact data model for a specific concept (e.g. "what is a Relationship") | `Ursatz-Library/Registry/*.md` (split by layer — see that folder's own index) |
| The full architectural spec behind a design decision | `Ursatz-Library/*_Specification.md` |
| Whether the *CLI* or *GUI* has built X, or a known bug in either | `Ursatz-Analyzer/Status.md` / `Known_Gaps.md`, `Ursatz-GUI/Status.md` / `Known_Gaps.md` |
| When/why something changed direction, or when a doc was last checked against code | `Reconciliation_Log.md` |
| Old superseded versions of any doc | `Archive/<date>/` |

## How this repo is organized

```
Overview/              cross-project only: vision, phase roadmap, business roadmap,
                        the design laws. Rarely restructured.
Ursatz-Library/         everything about the kernel: current status, known gaps, the
  Registry/             full data-model specs, and the System Registry (split by layer).
Ursatz-Analyzer/        current status + known gaps for the CLI consumer app.
Ursatz-GUI/             current status + known gaps for the GUI consumer app.
Ursatz-Composer/        placeholder — fill in using the same template once this repo exists.
Reconciliation_Log.md   append-only dated log of every verification pass and every
                        purpose/direction change. Never rewritten, only appended to.
Archive/                superseded doc versions, kept for history. Never edited.
```

## Working principles for this repo

1. **Every `Status.md` is a living document.** It describes current, verified-against-code
   state, not a point-in-time snapshot. It gets edited in place; history lives in git and in
   `Reconciliation_Log.md`, not in the filename.
2. **Purpose vs. implementation are separate sections** in each repo's `Status.md`. If a
   repo's goal changes, only the Purpose paragraph changes; implementation history isn't
   rewritten, and the change gets a dated entry in `Reconciliation_Log.md`.
3. **Adding a new repo to the ecosystem** (e.g. Ursatz-Composer) means filling in its folder
   using the same two-file template (`Status.md`, `Known_Gaps.md`), then adding one row to
   `Overview/Master_Roadmap.md`'s summary table. Nothing else needs to move.
4. **Don't trust a "Not Started"/"done" claim without a repo check.** This project's own
   history (see `Reconciliation_Log.md`) shows status claims lag real repo state often enough
   that it's standing practice to verify before relying on one, especially in the Registry.
5. **No file here references any repo's CLAUDE.md**, and no repo should be made to reference
   this repo. The two are kept independent by design.
