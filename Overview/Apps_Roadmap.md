# Music Apps Business Roadmap (Post-Ursatz)

**Last verified against repo state:** 2026-09-13.

**Scope note:** this document is product/market/phase sequencing — architecture belongs in
`Ursatz-Library/`, current build state belongs in each repo's `Status.md`. This document
answers "what are we building toward and why," not "is X built."

Begins where the Ursatz library ends. Ursatz (L0–L9) is the infrastructure layer —
theory-neutral computational music representation and analysis. Everything below is the
product layer built on top of it.

---

## Phase 1 — Analysis Engine

Narrow scope: pick one piece-type wedge (e.g. video game boss themes) and ship "here's
what's happening in this piece and why," end-to-end, for a real user. Proves the theory
engine produces genuine insight, not just data.

**Depends on:** Ursatz L0–L8-Transform (done). All planned L8-Analysis analyzers now have
real, tested implementations (Chord Identification incl. sevenths, Key Estimation, Voice
Leading, Cadence Detection/Classification, Motif Detection, Phrase Segmentation/Boundary
Detection, Period/Sentence/Form/Thematic Return Detection). No remaining technical blocker
to standing up the pipeline.

**The actual remaining gap:** Framework v1 verification against the fixed 5-piece test
corpus has still not been completed as a real pass — see `Overview/Master_Roadmap.md`'s
Phase 2.5 section. Two of the bugs that surfaced during the one partial (Bach-only) check
have since been fixed in code, but nothing has re-run the corpus to confirm, and 4 of 5
pieces have never been checked at all. Until that happens, "the pipeline exists" and "the
pipeline is verified correct" remain different, currently-conflated claims.

**Known gaps, not blockers (detail in `Ursatz-Library/Known_Gaps.md`):**
- Sonority construction now uses a real beat-grid/attack-based builder (fixed this changed
  since the original "exact-match rigidity" framing), but it's still a fixed quarter-note
  grid, not adaptive to texture or tempo.
- Voice Leading, Cadence Classification's PAC/IAC distinction, and real interval/motion
  computation are all still blocked on nothing actually consuming `TuningSystem`/
  `PitchRealization` yet, even though those types now exist in the kernel.
- No piece-type wedge (e.g. "video game boss themes") has been selected or scoped yet — this
  phase's narrow-scope framing is still aspirational.

## Phase 2 — Corpus / Provenance Library (GUI)

Not a standalone product — the library and visual layer on top of Phase 1's output. Serves
two roles: personal reference library, and the corpus Phase 3 will pull from.

**Build only** the storage/query shape Phase 1's analysis actually needs. Don't over-build
generic infrastructure ahead of real access patterns.

**Current state:** substantially further along than originally scoped. Ursatz-GUI now has a
full Library view (upload, rename, delete), a Collections grid (the corpus tool, no longer
just a decorative sidebar entry), and a tabbed Analysis view with claim detail, Evidence/
Provenance panels, and a Score tab rendering notation with claim-driven note highlighting.
See `Ursatz-GUI/Status.md` for the current feature list — this roadmap intentionally doesn't
duplicate it.

**Remaining Phase 2 work:** a real provenance lineage view (once Deliverable K / the
Provenance spec has a real implementation, not just the reference type); Accept/Dispute
claim actions are present in the UI but disabled (no promotion endpoint yet).

## Phase 3 — Generative Composition (Flagship)

"Blend the whimsical aspect of this with the darkness of that, match project requirements,
apply my style, generate the piece." Motif-up or form-down, either direction.

This is the point where composing becomes possible — new tracks, YouTube income, sheet music
sales. Biggest payoff, longest runway, hardest technical bar.

**Depends on:** Phases 1–2 working, including real Framework v1 verification (not yet done);
a Theory Framework and StyleModel/StatisticalModel that are real, not stubbed (`Constraint`/
`Preference` remain deliberate kernel stubs; no `StatisticalModel`/`StyleModel` exists yet).
A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult, L9,
music-query) does not exist yet — analysis output is queryable-by-construction (structured
Interpretation claims, real Relationship predicates) but nothing queries it yet.

No repo exists for this yet. Not broken down step-by-step — premature until Phase 1's
verification gap is closed.

## Phase 4 — Theory-Aware Synth

A synth that ties sound to musical function/role (via L6/L7 interpretation output) rather
than manipulating sound directly. Companion tool to personal composition workflow, not a
general product.

**Starts after:** Phase 3 is working and in active use.

## Phase 5 — New Instrument (Digital/Physical)

No defined scope or timeline. Parked.

---

## Governing principle

Each phase is scoped to the minimum needed to unblock the next. Ursatz technical work
(Registry, layer specs) stays in `Ursatz-Library/`. This document — product, market, phase
sequencing — stays separate from that architecture.
