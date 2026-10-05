# Music Apps Business Roadmap (Post-Ursatz)

**Last verified against repo state:** 2026-10-05.

**Scope note:** this document is product/market/phase sequencing; architecture belongs in
`Ursatz-Library/`, current build state belongs in each repo's `Status.md`. This document
answers "what are we building toward and why," not "is X built."

It begins where the Ursatz library ends. Ursatz (L0–L9) is the infrastructure layer,
theory-neutral computational music representation and analysis. Everything below is the
product layer built on top of it.

---

## Phase 1 — Analysis Engine

Narrow scope: pick one piece-type wedge (e.g. video game boss themes) and ship "here's
what's happening in this piece and why," end-to-end, for a real user. Proves the theory
engine produces genuine insight, not just data.

**Depends on:** Ursatz L0 to L8 (done) and the theory rule libraries (done, PRs #49 to #97:
tonality, harmony, counterpoint, form, meter). The analyzers exist, and the rules to judge
their output exist.

**The actual remaining gap: perception.** Nothing yet turns imported notes into the labeled
facts the rules need. Chord identification resolves only 61 of 394 beats of Field Nocturne
No. 1, MIDI import ignores meter, and the rule libraries aren't connected to the analysis
pipeline. Until that is fixed, Framework v1 corpus verification (4 of 5 pieces never
checked) can't meaningfully pass. Plan: `Overview/Master_Roadmap.md`, Phase 2.7.

**Known gaps, not blockers (detail in `Ursatz-Library/Known_Gaps.md`):**
- PAC vs. IAC, voice leading and motif detection still take caller-supplied data; the
  numeric pitch path (`PitchNumericValue`) exists but no analyzer derives soprano or bass
  lines from it yet.
- No piece-type wedge (e.g. "video game boss themes") has been selected or scoped yet; this
  phase's narrow-scope framing is still aspirational.

## Phase 2 — Corpus / Provenance Library (GUI)

Not a standalone product; it's the library and visual layer on top of Phase 1's output.
It serves two roles: personal reference library, and the corpus Phase 3 will pull from.

**Build only** the storage/query shape Phase 1's analysis actually needs. Don't over-build
generic infrastructure ahead of real access patterns.

**Current state:** substantially further along than originally scoped. Ursatz-GUI now has a
full Library view (upload, rename, delete), a Collections grid (the corpus tool, no longer
just a decorative sidebar entry), and a tabbed Analysis view with claim detail, Evidence/
Provenance panels, and a Score tab rendering notation with claim-driven note highlighting.
See `Ursatz-GUI/Status.md` for the current feature list; this roadmap intentionally doesn't
duplicate it.

**Remaining Phase 2 work:** a real provenance lineage view (once Deliverable K / the
Provenance spec has a real implementation, not just the reference type); Accept/Dispute
claim actions are present in the UI but disabled (no promotion endpoint yet).

## Phase 3 — Generative Composition (Flagship)

"Blend the whimsical aspect of this with the darkness of that, match project requirements,
apply my style, generate the piece." Motif-up or form-down, either direction.

This is the point where composing becomes possible, opening new tracks, YouTube income, and
sheet music sales. Biggest payoff, longest runway, hardest technical bar.

**Depends on:** Phases 1–2 working, including real Framework v1 verification (not yet done),
and a ranking layer that is real, not stubbed: `Constraint`/`Preference` evaluation remains a
deliberate kernel stub, and `StatisticalModel`/`StyleModel` exist as shapes only, with no
fitting. The framework rule libraries are the natural source of real constraints once they
are connected.
A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult, L9,
music-query) does not exist yet; analysis output is queryable-by-construction (structured
Interpretation claims, real Relationship predicates), but nothing queries it yet.

No repo exists for this yet. It's not broken down step-by-step; that would be premature
until Phase 1's verification gap is closed.

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
(Registry, layer specs) stays in `Ursatz-Library/`. This document, covering product, market,
and phase sequencing, stays separate from that architecture.
