# Music Apps Business Roadmap (Post-Ursatz) v3

**Status:** This roadmap begins where the Ursatz library ends. Ursatz (L0–L9) is the infrastructure layer — theory-neutral computational music representation and analysis. Everything below is the product layer built on top of it.

Supersedes v2 — Phase 1's "2 of 8 planned analyzers exist" is now stale. All 8 planned analyzers (Chord Identification, Key Estimation, Voice Leading, Cadence Detection, Motif Detection, Phrase Segmentation, plus the newly-added Period/Sentence/Form-level analyzers not originally counted in that 8) have real, tested implementations as of the Framework-stage session (2026-09-09). See Master Roadmap v5 for the full branch-by-branch breakdown. This does not mean Phase 1 is complete — see the new "Framework v1 verification" note below.

---

## Phase 1 — Analysis Engine

Narrow scope: pick one piece-type wedge (e.g. video game boss themes) and ship "here's what's happening in this piece and why," end-to-end, for a real user. Proves the theory engine produces genuine insight, not just data.

**Depends on:** Ursatz L0–L8-Transform (done). L8-Analysis: Chord Identification (now including seventh chords), Key Estimation, Voice Leading (packaging-only — see gap below), Cadence Detection/Classification, Motif Detection, Phrase Segmentation/Boundary Detection, Period Detection, Sentence Detection, Form Structure/Thematic Return Detection all implemented and unit-tested against the common-practice-minimal framework. No remaining technical blocker for standing up the pipeline.

**Framework v1 verification — not yet done, this is the actual remaining gap for Phase 1:** a fixed 5-piece test corpus (Bach BWV Anh. 114, Clementi Op. 36, Field Nocturne No. 1, Field Nocturne No. 5, Mozart K.545) is registered in `tests/fixtures/framework-v1-corpus/`, but the existing CorpusRegression test only confirms the 5 files are registered — nothing has yet run them through the full analysis pipeline and checked the output, either as an automated regression test or as a manual GUI spot-check. Until that happens, "the pipeline exists" and "the pipeline is verified correct" are different, currently-conflated claims.

**Known gaps, not blockers:**
- Chord-region matching has exact-match rigidity (Phase 2.5's first item) — confirmed during this session to extend to seventh-chord matching too, since it inherits the same mechanism by construction.
- Voice Leading, Cadence Classification's PAC/IAC distinction, and any real interval/motion computation are all blocked on `TuningSystem`/`PitchRealization` (Ursatz kernel, not yet built) — these analyzers currently only correctly *package* caller-supplied values, they don't compute real values from raw pitch data themselves. This is a concrete, not theoretical, blocker on analysis quality for real MIDI input.
- No piece-type wedge (e.g. "video game boss themes") has been selected or scoped yet — this phase's narrow-scope framing is still aspirational.

## Phase 2 — Corpus / Provenance Library (GUI)

Not a standalone product — the library and visual layer on top of Phase 1's output. Serves two roles: personal reference library, and the corpus Phase 3 will pull from.

**Build only** the storage/query shape Phase 1's analysis actually needs. Don't over-build generic infrastructure ahead of real access patterns.

Current state: persistence (SQLite-backed, NoteEvent/Uncertainty/Interpretation/Chord/Key) is merged. ursatz-gui backend implements the full Analyzer API (upload/list/detail); frontend is now wired to all three endpoints (list, detail, upload). Corpus tool (feat/corpus-tool) merged, including the framework-v1-reference-set registration. Remaining Phase 2 work: fix a known JSON-truncation bug in the detail endpoint, and (later) a real provenance lineage view once Deliverable K exists.

## Phase 3 — Generative Composition (Flagship)

"Blend the whimsical aspect of this with the darkness of that, match project requirements, apply my style, generate the piece." Motif-up or form-down, either direction.

This is the point where composing becomes possible — new tracks, YouTube income, sheet music sales. Biggest payoff, longest runway, hardest technical bar.

**Depends on:** Phases 1–2 working, including Framework v1 verification (not yet done — see above); Theory Framework and StyleModel/StatisticalModel real (not stubbed — `Constraint`/`Preference` are currently deliberate kernel stubs, and no `StatisticalModel`/`StyleModel` implementation exists yet). No PitchIdentity blocker remains. A Generator-facing query layer (PatternMatching/SimilarityModel/SearchResult, L9, music-query) does not yet exist — analysis output is queryable-by-construction (structured Interpretation claims, per the Framework Output Contract session) but nothing has been built to actually query it yet.

## Phase 4 — Theory-Aware Synth

A synth that ties sound to musical function/role (via L6/L7 interpretation output) rather than manipulating sound directly. Companion tool to personal composition workflow, not a general product.

**Starts after:** Phase 3 is working and in active use.

## Phase 5 — New Instrument (Digital/Physical)

No defined scope or timeline. Parked.

---

## Governing Principle

Each phase is scoped to the minimum needed to unblock the next. Ursatz technical work (Registry, layer specs) stays in the Ursatz project. This document — product, market, phase sequencing — stays separate from that architecture.

## Process note (added this session)

Registry/Roadmap status has repeatedly lagged actual repo state — observed on the majority of branches in the 2026-09-09 Framework-stage session (structures-minimal, semantics-minimal, and others all found "Not Started" items already built). Treat any "Not Started" status in these documents as unverified until a branch's own investigation step confirms it, not as ground truth by default.