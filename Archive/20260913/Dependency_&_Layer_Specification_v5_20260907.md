# DEPENDENCY & LAYER SPECIFICATION v5

Source: TDD v4 §190–192, System Registry v7.
Supersedes: v4. Reverts an incorrect v4 change (§Status, below) and records the confirmed
Relationship-based multi-part claim pattern used throughout the 2026-09-09 Framework-stage
session.

## Status

Both open issues from v2 remain resolved:
- **Interpretation-family → Framework ordering** — resolved via `FrameworkReference` (L6 holds a reference, resolution happens at L7+). Registry v3 closes this.
- **Theory → Corpus ordering** — confirmed already correctly handled: Theory-layer statistical objects (Tendency, StatisticalModel, StyleModel) consume Corpus data only as opaque pre-computed values, never importing the Corpus module itself.

**Correction (2026-09-09):** v3 of this document recorded a "Framework Output Contract
decision" relocating Motif, Cadence, Theme, and PhraseOccurrence from L5 (music-structure)
to L6 (music-interpretation), reasoning that their intended fields (FrameworkReference,
Context, Evidence) were L6 types and their L5 placement was therefore an undocumented
forward dependency. That decision was made without checking actual repo state first, and
was **wrong**: all four types already existed, committed, as plain L5 shells (id +
EntityID-reference-list, detection logic deliberately deferred to L8) — not as
fixed-vocabulary structs with L6-typed fields at all. There was no forward dependency to
resolve. This is reverted in full; see Layer contents below, which now matches the original,
correct v2/early-v3 placement.

The **actual** resolution, confirmed by building against it across the full Framework-stage
branch sequence (Cadence Detection, Motif Detection, Phrase Boundary Detection, Period
Detection, Sentence Detection, Form Structure Detection, Thematic Return Detection — all
merged): a framework-relative claim about an L5 shell is expressed as the existing generic
`Interpretation` type (L6, the same type Chord/Key already use), with its `target` set to
the L5 shell's EntityID. No new named Interpretation-family type was needed for any of
these. Where a claim links *two or more* L5/L6 elements (e.g. antecedent+consequent,
statement+repetition+continuation, a section boundary, a thematic return), the mechanism is
a `Relationship` (L6, music-relationship — see below) linking them, with the `Interpretation`
targeting the Relationship's id instead of a single element. This two-tier pattern (single
target → direct Interpretation; multi-element claim → Relationship + Interpretation on it)
is now the standing default for any future Framework-stage work.

**Relationship (L6, music-relationship) — confirmed implemented and in real use**, contrary
to its Registry status at the start of this session. Predicates actually used so far:
`precedes` (sequential order — Period antecedent→consequent, Form section order),
`transformed_from` (derivation — Sentence statement→repetition→continuation, direction:
derived-part → source), `resembles` (recurrence with no claimed derivation direction — Form
thematic return, Theme-to-Theme). This is the first real evidence the documented predicate
vocabulary works as designed; extend it before inventing new predicate names for a genuinely
new relationship kind.

**Relationship persistence — no repository home yet, stopgap in use (2026-09-09):**
`RelationshipRepository` does not exist in Ursatz's persistence layer (Registry: Not
Started). ursatz-gui's Framework-analyzer wiring (PR #13) needed to persist real
Relationship instances (Period/Sentence/Form/Thematic Return output) and could not wait on
this, so it added an ursatz-gui-owned SQLite table mirroring the existing
`kIndexTableName`/`kMetaTableName` pattern in `analysis_service.c`. This is
application-layer, outside the numbered stack, same as any other Persistence module (see
§Cross-cutting modules) — not a violation of layer rules, but also not the eventual
Ursatz-side `RelationshipRepository`. Revisit when/if a real `RelationshipRepository` is
built: ursatz-gui's stopgap table should be retired in favor of it, not left running in
parallel.

This specification is the working dependency contract.

---

## Layer order

```
L0  Identity
L1  Core
L2  Time / Pitch
L3  Events
L4  Containers
L5  Structures
L6  Context / Semantics
L7  Theory
L8  Analysis / Generation / Transformation
L9  I/O / Applications
```

Cross-cutting (attach to any layer, dependency direction governed by §Cross-Cutting Modules, not layer order): Provenance, Validation, Security, Observability, Persistence.

---

## Dependency rule

A module in layer **N** may depend only on modules in layers **0..N**, plus any cross-cutting module it's permitted to depend on. A module in layer **N** shall never depend on any module in layers **N+1..9**.

- A numbered layer MAY depend on a cross-cutting module, subject to that module's individual ceiling (below) — the per-module ceiling governs over this general permission.
- A cross-cutting module MUST obey its own stated ceiling and MUST NOT depend on any numbered layer above it. "Cross-cutting" describes *who may use it*, not *what it may use*.
- Persistence is outside the numbered domain stack entirely, I/O-adjacent — not "layer 9," not exempt from restriction.

---

## Layer contents (module → layer)

| Layer | Modules |
|---|---|
| L0 Identity | music-identity (EntityID, Version, VersionModel, SchemaVersion) |
| L1 Core | music-core (Entity, Value, Occurrence, the five equality notions, Canonical Musical Model/IR) |
| L2 Time/Pitch | music-time, music-pitch |
| L3 Events | music-events (Event, NoteEvent, RestEvent, ControlEvent, Tie, Slur, Articulation, Dynamics, Instrument*, InstrumentCapability*) |
| L4 Containers | music-containers (Container, Voice, Layer, Staff, Part, Measure, Phrase-container, Section, Movement, Work, Edition, Arrangement, Revision, SourceVariant) |
| L5 Structures | music-structure (Structure, Motif, MotifOccurrence, Figure, Theme, ThemeOccurrence, Cadence, PhraseOccurrence) |
| L6 Context/Semantics | music-context, music-semantics (Context, Interval, Sonority, Scale, ScaleDegree, Contour, RhythmicPattern — Contour/RhythmicPattern implemented), music-relationship (implemented — predicates in real use: precedes, transformed_from, resembles), music-interpretation (Assertion, Observation, Hypothesis, Interpretation, Evidence, Annotation, Chord, Key, ScholarlyAnnotation) |
| L7 Theory | music-theory, music-rules (Framework, Rule, Constraint, Preference, Heuristic, Tendency, StatisticalModel, StyleModel, Species/Common-Practice/advanced frameworks) |
| L8 Analysis/Generation/Transformation | music-analysis (Chord Identification incl. sevenths, Key Estimation, Voice Leading, Cadence Detection/Classification, Motif Detection, Phrase Segmentation/Boundary Detection, Period/Sentence/Form/Thematic Return Detection — all implemented), music-generation, music-transform (Analyzer, Procedure, Candidate, Intent, GenerationPlan, Generator, Transformation and subtypes) |
| L9 I/O/Applications | music-io, music-notation, music-performance, music-corpus, Query interface, Applications |

`*` = provisional placement — see below.

---

## Cross-cutting modules

| Module | Dependency ceiling | May be depended on by | Notes |
|---|---|---|---|
| Provenance | L0–L1 only | Any layer needing lineage (L1+) | MUST NOT depend on any layer above Core. No layer's own semantics may depend on Provenance to define correctness — it records lineage, not domain state. |
| Observability | L0 (EntityID) only | Any layer | MUST NOT alter domain semantics. |
| Security (plugin contract, sandboxing, query safety limits) | L0–L1 only | L7–L9 (plugins, queries) | MUST NOT be bypassed by any layer it governs. |
| Validation | Own layer + all lower layers | All layers | A validator for layer N depends on layer N plus everything N may depend on. MUST NOT depend upward. |
| Persistence (repositories, Cache) | Canonical IR / L1 only | I/O and application code outside the numbered stack | Outside the numbered stack. MUST NEVER be imported by L0–L8. See music-relationship note above re: ursatz-gui's stopgap Relationship-persistence table (2026-09-09) pending a real RelationshipRepository. |

No cross-cutting module may become an escape hatch around the layer architecture — routing through one doesn't exempt a dependency from the rules above.

**Provisional layer placements** (kept as-is, flagged as dependency-magnet risk):
- **Instrument/InstrumentCapability (L3)** — referenced by Events, Performance, Generation constraints, and Notation; plausible candidate for its own boundary once those relationships are fully specified.

**music-corpus (L9) — placement RESOLVED, no longer provisional:**
`corpus` (L9) depends only on `identity` (L0) and `persistence` (L9) —
dependency-pure, sibling to persistence. Analyzer and Generator depend on
`corpus` directly as peers; neither routes through the other.

`corpus_stats` (new, L9-adjacent) is a separate module, sibling to `corpus`,
depending on `corpus`, `persistence`, `interpretation`, and `identity`. This
split exists because chord-level statistics require reading `interpretation`
data, which `corpus` itself must not depend on (guard-tested in
tests/corpus/test_architecture_corpus.c). `corpus_stats` has its own
architecture guard (test_architecture_corpus_stats.c) permitting the wider
include set.

---

## Explicit prohibitions

**Layer leakage:**
```
Core MUST NOT import Theory | Pitch MUST NOT import Harmony/any Framework
Time MUST NOT import Notation
Events / Containers MUST NOT import Structures, Context, Theory, Analysis, I/O
Domain (L0–L8) MUST NOT import Persistence or SQL/storage tech
Plugins MUST NOT bypass capability/security boundaries
```

**Canonical IR authority:** the Canonical Musical Model/IR (L1) is the authoritative theory-neutral representation. Derived layers (L5–L9) may interpret, analyze, transform, render, project, or generate from it, but MUST NOT silently redefine canonical semantics:
```
Theory MUST NOT mutate Canonical Events
Notation / Performance MUST NOT redefine Canonical Events
```

**Semantic-leakage prohibitions:**
```
Analysis MUST NOT introduce theory-specific semantics into the Canonical IR
Generation MUST NOT require a specific Theory Framework unless explicitly selected
  by the GenerationPlan or equivalent configuration
Query projections MUST NOT become authoritative domain state
```
Framework-specific interpretation belongs in L6/L7, never embedded into L1–L3 canonical event semantics; query Views are projections, never a second authoritative representation.

**Pitch numeric semantics (confirmed this session, not previously stated explicitly here):**
```
L5-L8 analyzers MUST NOT compute real interval/tuning values (e.g. "is this a perfect
  fifth", semitone distance) directly from PitchIdentity — it is deliberately opaque
  (Law 18: asserts nothing about tuning, cardinality, or frequency). Such computation is
  blocked on TuningSystem/PitchRealization (L2, not yet built) at the kernel level, or must
  happen in a consumer application outside the kernel (e.g. ursatz-analyzer's
  pitch_class_util, which already does its own 12-TET-specific math for Chord/Key pipelines
  as an application-level concern, not a kernel one).
```
Confirmed as a real, not theoretical, constraint on Voice Leading, Cadence Classification's
PAC/IAC distinction, and Motif Detection this session — all three only operate on
caller-supplied or Contour/RhythmicPattern-derived (direction/duration-only) data.

---

## Registry-first dependency rule

The System Registry is an architectural authority, not documentation:

> Every inter-module dependency MUST be represented in the Registry before implementation. Architecture tests SHALL reject dependencies not authorized by the Registry.

Undocumented sibling coupling is prohibited at any layer, including within the same layer. A needed dependency not yet in the Registry must be reviewed and added there first — never introduced informally in code and reconciled later.

**Verify before deciding, not just before implementing (added this session):** a Registry-level
placement decision (like the incorrect L5→L6 relocation this document reverts above) is only
as good as the repo-state check behind it. Confirm actual committed state before making an
architecture decision, not only before a branch starts implementing one.

---

## Resolved issues (formerly Open Issues, v2 §7)

### Interpretation-family ↔ Framework ordering — RESOLVED
Registry previously listed `Interpretation`/`Chord`/`Key`/`ScaleDegree` (L6) as depending directly on `Framework` (L7) — a forward dependency, disallowed under strict layer order. Resolved via **option 1**: these types depend only on a `FrameworkReference` (ID/version) living at L6; resolution to the actual Framework happens at L7+. Closed by Registry v3.

### Theory (L7) ↔ Corpus (L9) ordering — RESOLVED
Registry listed `Tendency`/`StatisticalModel`/`StyleModel` (L7) as depending on `Corpus`/`Corpus Statistics` (L9) — also a forward dependency. Confirmed resolved in practice: Theory-layer statistical objects consume corpus data only as opaque pre-computed values, never importing the Corpus module itself (consistent with resolution option 1/3 from v2 — Corpus pushes derived statistics down rather than Theory pulling the module up). The `music-corpus` (L9) placement itself is also resolved — see "music-corpus placement" above — and is no longer an open question.

### L5 Structure-family placement — RESOLVED (corrected v3 error)
See §Status above. Motif, MotifOccurrence, Cadence, Theme, ThemeOccurrence, PhraseOccurrence,
Figure all confirmed at L5 as originally committed. No forward dependency existed; the v3
"fix" was solving a problem that wasn't real. Framework-relative claims about these L5 shells
use the existing generic Interpretation (L6) or Relationship+Interpretation (L6) mechanisms
instead of new named types.

---

## L9 detail

L9 is external/application-facing. Being in L9 doesn't make any of these authoritative over the canonical domain model (L1):

| Sub-area | Modules | Role |
|---|---|---|
| I/O adapters | music-io (MIDI/MusicXML/MEI/ABC importers & renderers, AudioAnalyzer, OCRImporter, LossModel) | Converts between external formats and Canonical IR. Never defines domain semantics. |
| Notation/rendering | music-notation | Representation/rendering only. Non-authoritative over Canonical Events. |
| Performance realization | music-performance | Realization only. Non-authoritative over Canonical Events. |
| Corpus/statistical infrastructure | music-corpus, corpus_stats | Derived, non-authoritative statistical knowledge. Placement resolved — see "music-corpus placement" above. |
| Query interfaces | Query Interface, PatternMatching, SimilarityModel, SearchResult, Views | Projections over Canonical IR. Must not become authoritative domain state. Not yet built — Framework-stage analyzer output is structurally queryable by construction (real Relationship predicates, structured Interpretation claims) but nothing queries it yet. |
| Applications | Composition, Practice, Ear Training, Analysis, Corpus, OCR, Audio, Notation, Performance apps | Consume domain services. Must not implement duplicate music semantics. |

---

## Architecture test mapping

| Test | Enforces |
|---|---|
| core cannot import theory | L1 ⇏ L7 |
| pitch cannot import harmony/frameworks | L2 ⇏ L7 |
| time cannot import notation | L2 ⇏ L9 |
| domain cannot import persistence | L0–L8 ⇏ Persistence |
| domain cannot import SQL/storage tech | L0–L8 ⇏ storage tech |
| theory cannot mutate canonical events | L7 write-access to L1–L3 = forbidden |
| notation/performance cannot redefine canonical events | L9 write/redefine access to L1–L3 = forbidden |
| analysis cannot introduce theory-specific semantics into Canonical IR | L8 writes to L1 restricted to theory-neutral fields |
| query projections cannot become authoritative domain state | Query/View modules read/derive only |
| plugins cannot bypass capability/security boundaries | L7–L9 plugin code ⇏ Security bypass |
| undocumented inter-module dependencies rejected | Any edge not in the Registry fails CI |

Per the Ursatz library's own architecture-guard tests (18+ per-module, regex-scanning `#include` lines against an allowlist), the core layer-leakage checks above are implemented and passing. This session's new analyzers (Cadence, Motif, Phrase Boundary, Period, Sentence, Form, Thematic Return Detection) each shipped with their own guard-test coverage confirming no upward/sideways leakage was introduced.

---

## Normative language key

- **MUST/SHALL** — mandatory architectural requirement.
- **SHOULD** — recommendation, deviation requires justification.
- **MAY** — permitted, not required.
- **provisional** — placement not yet frozen; may change.

This document does not elevate illustrative Registry examples (rows marked "Reference Concept"/"No" under Normative?) to new normative requirements beyond what the Registry states.