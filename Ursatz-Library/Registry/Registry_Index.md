# System Registry Index

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13). The Registry is the architectural
authority: every inter-module dependency must be represented here before implementation
(see `Dependency_Layer_Specification.md`'s Registry-first rule). Full status narrative for
what's built lives in `Status.md` / `Known_Gaps.md`; this Registry is the durable data-model
reference, not a status report, though each row does carry a Status field.

**Columns:** Name | Category | Purpose | Module | Depends On | Depended On By | Phase |
Status | Spec Ref | Normative? | Contract Type | Test Spec Ref

**Split by layer/section, to keep each file fast to open and edit:**

| File | Covers |
|---|---|
| `Registry_Foundations.md` | Identity/Versioning, Core Reference Types, Core/Value Objects, Temporal, Pitch, Events, Containers, Equality/Canonicalization, Canonical IR/Serialization/Persistence |
| `Registry_Structures_Semantics.md` | Structures/Occurrences, Semantic Layer, Relationship Model, Provenance |
| `Registry_Theory_Analysis.md` | Theory Frameworks, Analysis, Generation |
| `Registry_IO_Corpus_CrossCutting.md` | Query, Notation/Score, World/Source, I/O Adapters, Corpus, Plugin/Security, Validation, Testing, Work/Edition Model, Observability/Reproducibility, Performance/Recording, Transformation |

**Standing warning (earned the hard way, see `Reconciliation_Log.md`).** Treat any "Not
Started" status in this Registry as unverified until checked against real repo state. Status
lag has been the single most common documentation defect found in this project, so verify
before relying on a row, especially before making an architecture decision based on it.

## Change history

Full pre-2026-09-13 changelog (v2 through v7) is preserved in
`Archive/20260913/System_Registry_v7_20260907.md`. Going forward, changes to Registry content
are recorded in `Reconciliation_Log.md`, not as an in-file changelog. The Registry itself
should always describe current state only.
