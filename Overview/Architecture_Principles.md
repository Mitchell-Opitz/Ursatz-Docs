# Architecture Principles

**Last verified against repo state:** 2026-09-13. This document is conceptually stable;
it describes design law and layer shape, not current build status. Status lives in each
repo's `Status.md`.

This is the condensed, at-a-glance version. Full derivation and reasoning:
`Ursatz-Library/Technical_Design_Document.md` and `Ursatz-Library/Dependency_Layer_Specification.md`.

## Central principle

Distinguish what musical material *is* from what is *observed* about it, what is *inferred*,
how it's *interpreted*, under which *framework*, how it's *transformed*, and how it's
*rendered/performed*. No layer above the canonical model may silently redefine it. No
interpretation may silently become fact. No uncertainty may be discarded for downstream
convenience. No generated object may lose lineage. No external format may become the domain
model merely because it's convenient to serialize.

## The 25 Foundational Laws (normative)

1. Musical Material Is Not Theory — a framework may interpret material but not define its existence.
2. Representation Is Not Interpretation — a representation of an event doesn't imply its meaning.
3. Notation Is Not Musical Reality — notation is a representation/documentation of material.
4. Performance Is Not Symbolic Content — realization may differ from symbolic content.
5. Recording Is Not Performance Semantics — a recording is captured signal; meaning derived from it is observation/interpretation.
6. Relationships Are First-Class — important relationships aren't encoded solely as ad hoc fields.
7. Context Matters — context-dependent claims must explicitly identify that context.
8. Interpretation Is Non-Authoritative by Default — analytical claims don't silently overwrite source material.
9. Uncertainty Is Data — unknown, ambiguous, competing, and probabilistic information stays representable.
10. Provenance Is Preserved — derived objects retain lineage sufficient to determine how they were produced.
11. Theory Is Pluggable — the kernel doesn't depend on any particular music-theoretical framework.
12. Rules and Statistics Are Different — a statistical tendency never automatically becomes a theoretical rule.
13. Constraints and Preferences Are Different — a preference ranks candidates without invalidating them.
14. External Formats Are Adapters — MIDI/MusicXML/ABC/MEI/audio/DB/JSON/protobuf don't define the canonical domain.
15. Storage Is Not Domain Semantics — any storage technology remains replaceable.
16. Identity Is Not Position — array index, measure number, file offset, row number are never sole identity.
17. Occurrence Is Distinct from Conceptual Identity — a recurring motif and one occurrence of it are different concepts.
18. Written and Sounding Pitch Are Distinct — instrument transposition isn't embedded into the universal pitch definition.
19. Symbolic Time Is Exact — authoritative symbolic time doesn't depend on floating-point arithmetic.
20. Transformations Do Not Mutate Their Inputs — they create new versions or derived objects.
21. Stochastic Processes Are Reproducible — random generation supports explicit seeds and reproducibility metadata.
22. Loss Is Explicit — conversions that can't preserve information report the loss.
23. Canonicalization Is Not Equality — two objects may canonicalize equivalently without being identical entities.
24. Partial Information Is Valid Information — an incomplete object is representable when the domain permits incomplete knowledge.
25. The Foundation Remains Small — new concepts are introduced above the kernel whenever possible.

## Layer model (current, corrected)

```
L0  Identity
L1  Core
L2  Time / Pitch
L3  Events
L4  Containers
L5  Structures            (Motif, Cadence, Theme, PhraseOccurrence, etc. — plain shells)
L6  Context / Semantics   (Interpretation, Relationship — the claim mechanism)
L7  Theory                (Framework, Rule, Constraint, Preference)
L8  Analysis / Generation / Transformation
L9  I/O / Applications
```

Cross-cutting modules attach to any layer; their dependency direction is governed by their
own ceiling, not layer order: **Provenance** (L0–L1 ceiling), **Observability** (L0
ceiling), **Security** (L0–L1 ceiling), **Validation** (own layer + all lower), and
**Persistence** (Canonical IR/L1 only, outside the numbered stack entirely, never imported
by L0–L8).

**Dependency rule:** a module in layer N may depend only on modules in layers 0..N, plus any
cross-cutting module it's permitted to use, never upward. Full prohibitions list and the
Registry-first enforcement rule: `Ursatz-Library/Dependency_Layer_Specification.md`.

## The durable claim pattern (established 2026-09-09, do not re-litigate per branch)

A framework-relative claim about an L5 shell (Motif, Cadence, Theme, etc.) is a generic
`Interpretation` (L6) targeting the shell's EntityID, never a new named claim-type. Where a
claim links two or more elements (antecedent+consequent, a section boundary, a thematic
return), the mechanism is a `Relationship` (L6) linking them, with the `Interpretation`
targeting the Relationship's id instead. This is proven across Period, Sentence, Form, and
Thematic Return Detection and is the default assumption for any future analyzer work.

## Standing lesson (2026-09-09, why the Registry-first rule matters in practice)

A mid-session architecture decision relocated four L5 types to L6, reasoning from what their
fields *should* be rather than checking what was actually committed. It was wrong, and cost a
revert. **Verify actual repo state before making an architecture decision, not only before
implementing one.** This same discipline now also applies to documentation claims generally;
see `Reconciliation_Log.md`.
