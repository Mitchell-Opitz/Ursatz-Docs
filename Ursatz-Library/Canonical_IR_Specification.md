# Canonical IR Specification (Deliverable N)

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13). This is a Phase-1 kernel spec; Phase-1
objects are complete and unchanged since — nothing in the 2026-09-09 Framework Stage session
or since (through Ursatz PR #46) touched this layer. Content below is otherwise unchanged
from the prior version (previously "v3").

Source: Technical Design Document, System Registry, Dependency & Layer Specification,
Domain Specification.

## Status

The three reference types (`VoiceReference`, `RelationshipReference`, `ProvenanceReference`)
used throughout this document are Registry-authorized and implemented. Remaining open items:
the Registry module-dependency wording for the Canonical IR row itself (see Open Issues),
NoteEvent's category boundary, TimeSpan's boundary vocabulary, and the general
partial-information model beyond ProvenanceReference.

## Purpose

Answers: **what is the authoritative machine representation of musical information?**

The Canonical IR is the theory-neutral, serialization-independent, storage-independent
object graph representing musical material (Laws 14/15). It is:
- **not** a wire format (Deliverable O);
- **not** a database schema (Deliverable P);
- **not** a notation document, performance recording, or audio signal (Laws 3/4/5) — those
  reference canonical objects but aren't canonical themselves;
- **not** owned by any theory framework (Laws 1/11) — frameworks consume it.

Every layer above (analysis, generation, transformation, I/O, applications) reads from and
writes to the Canonical IR. No other representation is authoritative.

## Open issue: what module is "the Canonical IR"?

"Canonical IR" throughout this document names the authoritative object graph and its
semantics — never a single importable code module. The System Registry lists
`Canonical Musical Model / IR` at module `music-core`, layer 1, with
`Depends On: Event, Structure, Relationship, Context, Provenance` — taken literally, this
would mean L1 depends on L3/L5/L6, violating the Dependency Spec outright.

**Working interpretation used in this document (not an architectural resolution):**
"Canonical IR" is the *emergent graph* formed when L1 base categories and reference types
combine, at consumption time, with higher-layer objects that reference them. No single
module imports all of it; applications and query code assemble the graph by resolving
references across layers. Under this reading, the Registry's "Depends On" column describes
conceptual composition, not an import edge — but the Registry row itself remains unamended,
and the literal contradiction is a real open item until it's annotated as such. Every graph
diagram below should be read as describing composition, not a module's import list.

## What the IR is built from

A graph of typed nodes (Entities, Values, Occurrences, Events, Containers, Structures)
connected by typed edges (Relationships, and lighter-weight reference types), annotated with
Context, Interpretation, Evidence, Uncertainty, and Provenance.

| Category | Role | Base layer |
|---|---|---|
| Entity | Identifiable conceptual object (EntityID + Version) | L1 base type; concrete entities at various layers |
| Value | Immutable content, identity = value | L1 base type; concrete values at various layers |
| Occurrence | Situated instance of a conceptual object | L1 base type; concrete occurrences at L5 |
| Event | Occurrence tied to a temporal domain | L3 |
| Container | Groups/organizes objects | L4 |
| Structure | Higher-level semantic organization, references events without duplicating | L5 |
| Relationship | First-class typed connection between graph nodes | L6 |
| Assertion/Observation/Hypothesis/Interpretation | Claims about graph nodes, non-authoritative by default (Law 8) | L6 |
| Context | Conditions under which a claim is evaluated | L6 |
| Provenance | Lineage of derived nodes (Activity/Agent/Derivation) | Cross-cutting, L0–L1 ceiling |

No category is collapsed into another (Laws 3–5): a NoteEvent is never treated as a notation
symbol, performed sound, or chord member for convenience.

## Identity and reference model

Every persistent node is addressed by `EntityID` (L0), stable across revisions, carrying no
positional meaning (Law 16). Two connection mechanisms:

- **Relationships (L6):** full first-class edges with predicate, scope, context, evidence,
  provenance — used where the connection itself carries semantic weight (`derived_from`,
  `resembles`, `contrasts_with`). **In real use since 2026-09-09** — see
  `Ursatz-Library/Status.md`.
- **Opaque references** (`VoiceReference`/`RelationshipReference` at L1, `ProvenanceReference`
  cross-cutting L0–L1): lightweight EntityID-shaped pointers used where a lower-layer node
  must point at a higher-layer node without importing that higher layer's module. Carry no
  embedded semantics of their target; resolution happens only at or above the target's own
  layer.

This is what makes the IR a **graph** rather than a tree of embedded objects — nodes point at
each other by identity, never by embedding.

## Phase-1 canonical graph

```
EntityID                    (L0)
Duration                    (L2, Value)
MusicalTime                 (L2, Value)
TimeSpan                    (L2, Value)
PitchIdentity                (L2, Value)
PitchSpelling                 (L2, Value)
NoteEvent                    (L3, Event — category boundary unresolved)
Voice                        (L4, Container)
VoiceReference                (L1, Value)
RelationshipReference          (L1, Value)
ProvenanceReference             (Cross-cutting L0–L1, Value)
```

```
NoteEvent
  ├─ id                    : EntityID
  ├─ temporal_anchor        : MusicalTime (via TemporalAnchor)
  ├─ duration               : Duration
  ├─ pitch                  : PitchIdentity
  ├─ optional_pitch_spelling: PitchSpelling
  ├─ optional_voice         : VoiceReference ──resolves-at-L4──▶ Voice
  └─ provenance             : ProvenanceReference ──resolves-at-cross-cutting──▶ Provenance record
```

`Voice` holds NoteEvent membership via references, not by embedding NoteEvent objects — the
edge is bidirectional in meaning (Voice groups notes; a note optionally names its voice) but
implemented as two independently-resolvable pointers, not shared mutable structure.

Later phases extend the same graph with Structure/Occurrence nodes (Motif, Phrase, Theme —
reference events without duplicating), Relationship edges (now real — Period/Sentence/Form/
Thematic Return Detection all produce them), Context/Interpretation nodes (Chord, Key,
ScaleDegree — attach to Sonority/Context, never mutate the underlying NoteEvents), and
Provenance chains — all following this same identity-and-reference pattern.

## Requirement compliance

| Requirement | How satisfied | Status |
|---|---|---|
| exact time | `MusicalTime` — exact rational, no float (Law 19) | Satisfied |
| exact duration | `Duration` — exact rational, `>= 0` invariant | Satisfied |
| multiple tuning systems | `PitchIdentity` makes no tuning assumption; `TuningSystem`/`PitchRealization` types now exist | Deferred — types exist, nothing consumes them yet (see `Status.md`) |
| multiple pitch representations | `PitchIdentity`/`PitchSpelling`/`PitchRealization` split (Law 18) | Partial — Realization type exists, unused |
| stable identity | `EntityID`, position-independent (Law 16) | Satisfied |
| partial information | `ProvenanceReference` three-state model; general model for other fields deferred | Partial |
| relationships | L6 `Relationship`; L1 `RelationshipReference` for lower-layer participation | Implemented and in real production use |
| contexts | L6 `Context`, composable/nested | Structurally defined; not yet implemented |
| provenance | `ProvenanceReference` + Provenance/Activity/Agent/Derivation | Structurally defined and Registry-authorized; persistence covers Uncertainty (partial) |
| versioning | `EntityID` + `Version` distinct | Structurally defined |
| extensions | Core Extension Rule | Policy defined, no mechanism implemented yet |

## Uncertainty and non-authoritative claims

The canonical graph (NoteEvent, Voice, etc.) holds only what is asserted as musical material.
Claims *about* that material — key, chord, scale degree, motif membership — are separate
node types (`Observation`, `Hypothesis`, `Interpretation`) referencing canonical nodes
without altering them (Law 8). Multiple competing Interpretations may coexist pointing at the
same Sonority; no promotion to canonical fact happens except through an explicit, tracked
action ("Promotion").

**Consequence: the canonical IR graph is never mutated by an analysis or interpretation
process.** Analysis/interpretation only adds new nodes and edges pointing at existing
canonical nodes. This doesn't conflict with ordinary entity lifecycle — a NoteEvent may still
be revised through its own lifecycle (creation → revision → derivation → supersession →
deletion/tombstoning), producing a new `Version` under the same `EntityID`. What's prohibited
is a downstream Interpretation/Analysis process rewriting canonical fields as a side effect
of producing a claim about them.

## Versioning

Identity (`EntityID`) and revision (`Version`) are distinct; the canonical IR has no single
global version/snapshot number — each Entity-category node is independently versioned. A
coherent multi-node snapshot ("state of this Work as of commit X") is a Corpus/Work-level
concern, out of scope here.

Derived nodes (Transformation/Generation output) receive **new** identities with
`ProvenanceReference(state=present)` pointing back to their sources (Law 20) — never
revisions of their input nodes.

## Explicitly excluded from the Canonical IR

Not canonical IR nodes, even though they reference canonical IR nodes (Laws 3/4/5):
- Notation documents (engraving, layout, graphic coordinates, editorial markings) — L9.
- Performance realizations (onset/release/velocity/timing deviation) — `PerformanceEvent`,
  L9, references NoteEvent via a 0..n `realizes` relationship, never redefines it.
- Recordings/audio signal data — `Recording`/`AudioTime`, L9.
- Any serialized byte stream — Deliverable O.
- Any storage row/document — Deliverable P.
- MusicXML export output (`src/export/musicxml_export.*`) — a rendering, not a canonical node.

A conversion or query treating any of the above as canonical fact is a Dependency Spec
violation ("MUST NOT redefine Canonical Events").

## Worked example: C4–E4–G4 triad

Three quarter notes. `optional_pitch_spelling`/`optional_voice` omitted as legitimately
optional; `provenance` is shown explicitly (never silently omitted), using `absent` for
hand-authored fixtures:

```
NoteEvent N1: { id: "note-1", temporal_anchor: 0/1, duration: 1/4, pitch: PitchIdentity(C4),
                provenance: ProvenanceReference(state=absent, absence_reason="hand-authored example fixture") }
NoteEvent N2: { id: "note-2", temporal_anchor: 0/1, duration: 1/4, pitch: PitchIdentity(E4),
                provenance: ProvenanceReference(state=absent, absence_reason="hand-authored example fixture") }
NoteEvent N3: { id: "note-3", temporal_anchor: 0/1, duration: 1/4, pitch: PitchIdentity(G4),
                provenance: ProvenanceReference(state=absent, absence_reason="hand-authored example fixture") }
```

These three nodes are the entirety of the canonical fact. Everything else is derived,
non-authoritative, and attaches by reference:

```
Interval(N1,N2)     -- derived, L6 semantic layer
Interval(N2,N3)     -- derived
Sonority S1 = {N1,N2,N3}   -- derived, selection semantics explicit

Interpretation A: { target: S1, claim: "C major triad", framework: tonal-harmony }
Interpretation B: { target: S1, claim: "VI in E minor", framework: tonal-harmony, context: E-minor }
```

Both interpretations coexist; N1/N2/N3 are never modified by either.

Transposition (+P5) produces new identities with provenance, never mutates N1–N3. Unlike
N1–N3, these are derived nodes, so `state=present` with a resolving `provenance_id` is
required (`absent` is not available for Activity-produced output):

```
NoteEvent N4: { id: "note-4", pitch: PitchIdentity(G4),
                provenance: ProvenanceReference(state=present, provenance_id="prov-transpose-1"), derived_from: N1 }
NoteEvent N5: { id: "note-5", pitch: PitchIdentity(B4),
                provenance: ProvenanceReference(state=present, provenance_id="prov-transpose-1"), derived_from: N2 }
NoteEvent N6: { id: "note-6", pitch: PitchIdentity(D5),
                provenance: ProvenanceReference(state=present, provenance_id="prov-transpose-1"), derived_from: N3 }
```

(`prov-transpose-1` resolves, at the cross-cutting Provenance layer, to a Derivation record
naming the `Transposition(+P5)` activity — the record itself is Deliverable K's scope, still
not written.)

## Extension mechanism

New musical concepts enter the canonical graph only under the Core Extension Rule
(framework-neutral, required by multiple subsystems, not representable as an extension,
stable semantics, no theory leakage). The default path for a new concept is **not** a new
canonical node type — it's one of:
- a new `Relationship` predicate;
- a new `Interpretation`/`Annotation` type referencing existing canonical nodes;
- a new `Structure`/`Occurrence` pair referencing existing events without duplicating them.

Only if none of these fit, and all five extension criteria are met, does a new base-category
node enter L1–L4.

## Open Issues

1. **Canonical IR module-dependency conflict.** The Registry's literal "Depends On" list for
   `Canonical Musical Model / IR` conflicts with strict layer ordering. Working interpretation
   (composition, not import) used in this document; Registry annotation to reflect that still
   needed.
2. **NoteEvent category boundary unresolved** — affects whether "Event" is the correct
   permanent category for the canonical graph's core node, or whether Entity/Occurrence
   absorbs it.
3. **TimeSpan boundary vocabulary provisional** — affects how Occurrence/Structure regions
   are represented.
4. **General partial-information model beyond ProvenanceReference is unscoped** —
   pitch/other-field partial knowledge (e.g. "pitch = unknown / pitch_candidates = [...]")
   has no worked node model yet; needed before Phase 4 (Observation/Hypothesis) can be
   specified against this IR.

## Resolved (no longer open)

- Reference-type Registry authorization (`VoiceReference`/`RelationshipReference`/
  `ProvenanceReference`) — resolved.
