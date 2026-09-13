\# Music Apps Business Roadmap (Post-Ursatz)

&#x20;

\*\*Status:\*\* This roadmap begins where the Ursatz library ends. Ursatz (L0–L9) is the

infrastructure layer — theory-neutral computational music representation and analysis.

Everything below is the product layer built on top of it.

&#x20;

\---

&#x20;

\## Phase 1 — Analysis Engine

Narrow scope: pick one piece-type wedge (e.g. video game boss themes) and ship

"here's what's happening in this piece and why," end-to-end, for a real user.

Proves the theory engine produces genuine insight, not just data.

&#x20;

\*\*Depends on:\*\* Ursatz L0–L7 (done/near-done), L8 real transform logic (blocked on

PitchIdentity arithmetic gap — must resolve before this phase can complete).

&#x20;

\## Phase 2 — Corpus / Provenance Library (GUI)

Not a standalone product — the library and visual layer on top of Phase 1's output.

Serves two roles: personal reference library, and the corpus Phase 3 will pull from.

&#x20;

\*\*Build only\*\* the storage/query shape Phase 1's analysis actually needs. Don't

over-build generic infrastructure ahead of real access patterns.

&#x20;

\## Phase 3 — Generative Composition (Flagship)

"Blend the whimsical aspect of this with the darkness of that, match project

requirements, apply my style, generate the piece." Motif-up or form-down, either

direction.

&#x20;

This is the point where composing becomes possible — new tracks, YouTube income,

sheet music sales. Biggest payoff, longest runway, hardest technical bar.

&#x20;

\*\*Depends on:\*\* Phases 1–2 working; Theory Framework and StyleModel/StatisticalModel

real (not stubbed); PitchIdentity gap resolved.

&#x20;

\## Phase 4 — Theory-Aware Synth

A synth that ties sound to musical function/role (via L6/L7 interpretation output)

rather than manipulating sound directly. Companion tool to personal composition

workflow, not a general product.

&#x20;

\*\*Starts after:\*\* Phase 3 is working and in active use.

&#x20;

\## Phase 5 — New Instrument (Digital/Physical)

No defined scope or timeline. Parked.

&#x20;

\---

&#x20;

\## Governing Principle

Each phase is scoped to the minimum needed to unblock the next. Ursatz technical

work (Registry, layer specs) stays in the Ursatz project. This document — product,

market, phase sequencing — stays separate from that architecture.

&#x20;

