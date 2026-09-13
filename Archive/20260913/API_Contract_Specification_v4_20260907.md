# API / CONTRACT SPECIFICATION — Software Interaction Layer v4

Source: Domain Ontology Specification v4, TDD v4 §124–125/132/138/147/153, Dependency & Layer Specification v3.
Scope: Phase-1 ontology objects (TDD §233).
Supersedes: v3. Condensed — resolved blockers and superseded-status narration removed. Contract content unchanged.

## Purpose

Deliverable B (Domain Spec) defines what the domain objects *are*. This document defines how code is permitted to construct, inspect, and serialize them — the language/framework-independent contract surface. It does not define storage (Deliverable P), wire formats (Deliverable O), or collection-level query semantics (Deliverable Q); those build on this.

## Status

`VoiceReference`, `RelationshipReference`, and `ProvenanceReference` contracts below are no longer blocked — Registry sync landed in Registry v3, so these are implementable, not proposed-only. Remaining open items inherited from Domain Spec v4: NoteEvent's category boundary, TimeSpan's boundary vocabulary.

---

## Contract categories

| Category | Phase-1 applicability |
|---|---|
| Constructors | All 10 objects + 3 reference types |
| Queries | All 10 objects + 3 reference types |
| Transformations | Primitive arithmetic only (§Primitive Arithmetic below); full Transformation Architecture (TDD §138–146, Deliverable M) is Phase 7 |
| Services | None — no Phase-1 object has a persistence contract yet (Deliverable P) |
| Analyzers | None — Phase 8, operates on Structures/Relationships not yet specified |
| Generators | None — Phase 12, operates on Intent/Constraint objects not yet specified |
| Serialization | All 10 objects + 3 reference types — abstract round-trip contract only; wire format is Deliverable O's scope |

---

## Cross-cutting rules

- **C1 Immutability** — every Value object's constructors produce a fully-formed, immutable instance; no setters anywhere.
- **C2 Entity revision, not mutation** — NoteEvent and Voice are never mutated in place. Identity (EntityID) stays stable across revisions; each revision is a new construction. NoteEvent's new construction additionally carries a ProvenanceReference linking to its source (Law 20, see C6). Voice's membership-revision mechanics remain deferred to Deliverable F.
- **C3 Fail-fast validation** — every constructor validates Domain Spec invariants synchronously and rejects on violation (`InvariantViolation`). No "construct now, validate later" path.
- **C4 Reference opacity** — constructors/queries touching a reference type operate only on the opaque EntityID-shaped value; no L0–L4 object may resolve a reference into its target. Resolution queries live separately (§Resolution below).
- **C5 No floating point** — no MusicalTime/Duration/rational-derived quantity is ever accepted, returned, or compared as float/double (Law 19).
- **C6 Provenance state is mandatory** — any constructor for an object carrying ProvenanceReference (Phase 1: NoteEvent) requires an explicit `present`/`absent` state; no silent omission. Only Activity-produced objects (e.g. Transformation output) must use `state=present` with a resolving `provenance_id`; hand-authored objects may legitimately use `state=absent`.

---

## Value object contracts (L0/L2)

### EntityID
```
EntityID.of(value: String) -> EntityID
  raises InvariantViolation if empty or positionally-encoded (best-effort check)
.value() -> String | .equals(other) -> Boolean | .hashCode() -> Integer
.serialize() -> opaque scalar | EntityID.deserialize(scalar) -> EntityID
  (MUST NOT accept row-number/array-index-shaped input)
```

### Duration
```
Duration.of(numerator: Integer, denominator: PositiveInteger) -> Duration
  raises InvariantViolation if denominator <= 0 or reduced value < 0
  postcondition: stored in reduced (lowest-terms) form
.numerator() / .denominator() / .equals(other) [reduced-value equality] / .compareTo(other)
.add(other: Duration) -> Duration | .subtract(other: Duration) -> Duration
  raises InvariantViolation if result < 0
.serialize() -> {numerator, denominator} (exact integers, never float) | .deserialize(data)
```

### MusicalTime
```
MusicalTime.of(numerator, denominator: PositiveInteger) -> MusicalTime
  raises InvariantViolation if denominator <= 0
.numerator() / .denominator() / .equals(other) / .compareTo(other)
.add(d: Duration) -> MusicalTime | .subtract(other: MusicalTime) -> Duration
  (MUST NOT accept/return a Duration where MusicalTime is expected, or vice versa)
.serialize() -> {numerator, denominator} | .deserialize(data)
```

### TimeSpan
Boundary vocabulary provisional (Domain Spec §TimeSpan) — this contract uses the working four-value enum and will be revised without notice-of-breaking-change when Deliverable C lands.
```
TimeSpan.of(start: MusicalTime, end: MusicalTime, boundary: BoundaryKind) -> TimeSpan
  raises InvariantViolation if end < start, or if boundary is omitted (no implicit default)
  -- BoundaryKind = { closed, open, left_closed, right_closed }  (working vocabulary only)
.start() / .end() / .boundary() / .equals(other) [start, end, AND boundary must all match]
.serialize() -> {start, end, boundary} (boundary MUST be explicit; deserializer MUST reject
  a payload missing boundary rather than default to closed) | .deserialize(data)
```

### PitchIdentity
Equivalence relation RESOLVED (v0.3): strict token identity, permanent by design.
```
PitchIdentity.of(token: opaque) -> PitchIdentity
  raises InvariantViolation if token is EntityID-typed or carries entity-lifecycle semantics
  (MUST NOT require tuning-system parameters)
.token() -> opaque (MUST NOT be exposed in an EntityID-mistakable form)
.equals(other) -> Boolean  -- strict token identity; theory-relative equivalence is PitchClass's job
.serialize() -> opaque (MUST NOT hard-code 12-TET/A440/MIDI) | .deserialize(data)
  -- token's concrete representation deferred to Deliverable D; this fixes only that
     serialization is opaque and non-EntityID-shaped
```

### PitchSpelling
```
PitchSpelling.of(letter: Letter, accidental: Accidental, octave: Integer) -> PitchSpelling
  raises InvariantViolation if letter not in declared alphabet
.letter() / .accidental() / .octave() / .equals(other) [(letter,accidental,octave) triple —
  C# and Db are NOT equal]
.serialize() -> {letter, accidental, octave} (all explicit, never reconstructed from
  PitchRealization) | .deserialize(data)
```

---

## Entity / Container / Reference contracts (L1/L3/L4)

### NoteEvent (L3)
Category boundary (Entity vs. Event vs. Occurrence) remains unresolved (Domain Spec). This contract is written against the current provisional Event-category treatment.
```
NoteEvent.create(id: EntityID, temporal_anchor: TemporalAnchor, duration: Duration,
  pitch: PitchIdentity, optional_pitch_spelling: PitchSpelling?,
  optional_voice: VoiceReference?, provenance: ProvenanceReference) -> NoteEvent
  raises InvariantViolation if duration < 0
  raises InvariantViolation if provenance state is not explicitly present/absent (C6) —
    only Activity-produced NoteEvents require state=present with a resolving provenance_id;
    hand-authored NoteEvents may use state=absent
  raises InvariantViolation if optional_voice is anything other than a VoiceReference
    (enforces music-events / music-containers non-dependency)
  -- id is caller-supplied; this contract validates and stores it, doesn't allocate identity

.id() / .temporalAnchor() / .duration() / .pitch() / .optionalPitchSpelling()
.optionalVoice() -> VoiceReference?  (returns the opaque reference; MUST NOT resolve it)
.provenance() -> ProvenanceReference
.equalsById(other) -> Boolean   -- identity equality, primary
.equalsByValue(other) -> Boolean -- structural equality; does NOT imply same event
-- no mutator; revision = construct a new NoteEvent chaining provenance back to this one

.serialize() -> {id, temporal_anchor, duration, pitch, optional_pitch_spelling?,
  optional_voice?, provenance}
  -- id as EntityID (never positional); optional_voice as opaque reference only;
     provenance with explicit present/absent marker
.deserialize(data) -> NoteEvent  (same validation as .create())
```

### Voice (L4)
```
Voice.create(id: EntityID) -> Voice
  -- membership is NOT a constructor argument (Domain Spec: no owned/mutable event list)
.id() / .equalsById(other)
-- Membership query/mutation contract is NOT specified here; deferred to Deliverable F
  (Container and Structure Specification). No Voice membership contract should be
  implemented against this document.
.serialize() -> {id} | .deserialize(data)
```

### Primitive value arithmetic (note)
`.add()`/`.subtract()` on Duration and MusicalTime are the *only* Phase-1 "transformation" contracts. They are not instances of the TDD §138 Transformation Architecture (Transposition, Inversion, Retrograde, etc.) — those operate on NoteEvent-and-above, require provenance linkage (Law 20), and belong to Deliverable M (Phase 7). Primitive rational arithmetic produces a new Value, not a derived Entity, so it needs no provenance.

### VoiceReference (L1)
```
VoiceReference.of(voice_id: EntityID) -> VoiceReference
.voiceId() / .equals(other) [value equality on referenced EntityID]
-- no resolve() method; resolution to a Voice performed only at L4+ (see Resolution below)
.serialize() -> opaque identifier only (no inlined Voice payload) | .deserialize(data)
```

### RelationshipReference (L1)
```
RelationshipReference.of(relationship_id: EntityID) -> RelationshipReference
.relationshipId() / .equals(other)
-- no resolve() method; resolution occurs only at L6+
.serialize() -> opaque identifier only | .deserialize(data)
```

### ProvenanceReference (cross-cutting L0–L1)
```
ProvenanceReference.present(provenance_id: EntityID) -> ProvenanceReference
ProvenanceReference.absent(absence_reason: String?) -> ProvenanceReference
  -- MUST NOT be used for output of any Activity

.state() -> {present, absent}  (never "omitted" — omission is not constructible)
.provenanceId() -> EntityID  (meaningful only when state=present; raises otherwise)
.absenceReason() -> String?  (meaningful only when state=absent)
.equals(other) -> Boolean  -- equality on (state, provenance_id) only; absence_reason
  excluded (flagged as revisable, Domain Spec)

-- no resolve() method; resolution to a Provenance/Activity/Agent/Derivation chain occurs
   at L1+, only when state=present

.serialize() -> {state, provenance_id?, absence_reason?}
  -- state MUST always serialize explicitly; provenance_id MUST NOT serialize when state=absent
.deserialize(data) -> ProvenanceReference
  raises InvariantViolation if state is missing from payload (never defaults to absent)
```

---

## Resolution query contract (abstract, out of Phase-1 scope)

Reference types deliberately expose no `resolve()` method themselves — resolution is performed by code above the target's layer:
```
VoiceResolver.resolve(ref: VoiceReference) -> Voice | DanglingReference          -- L4+
RelationshipResolver.resolve(ref: RelationshipReference) -> Relationship | DanglingReference  -- L6+
ProvenanceResolver.resolve(ref: ProvenanceReference) -> ProvenanceRecord | DanglingReference   -- L1+
```
A dangling reference on resolution is an invariant violation, not merely a null result — callers must be able to distinguish "absent" from "dangling." Whether a reference to a tombstoned target counts as dangling is left to the eventual lifecycle/resolution specs (Deliverable G for Relationship, K for Provenance, Container/Structure Spec for Voice).

---

## Error contract (shared)

```
InvariantViolation — raised synchronously by any constructor/deserializer whose input would
  produce an object violating a Domain Spec invariant. Construction MUST NOT succeed and be
  flagged invalid later.

DanglingReference — raised by a resolution query (above) when a reference's target cannot be
  resolved. Not raised by any constructor/query contract in this document, since none of those
  perform resolution.
```
No other error type is contracted at Phase 1. Object-specific error subtypes are an implementation detail this document doesn't mandate.

---

## Out of scope at Phase 1

```
Services   — repository/persistence interfaces (TDD §129). Owning deliverable: P.
Analyzers  — TDD §148. Phase 8 (TDD §242).
Generators — TDD §160. Phase 12 (TDD §246).
Query Architecture (collection-level pattern matching/search, TDD §132–137) — Deliverable Q.
```
None of the ten Phase-1 objects are inputs to a Service, Analyzer, or Generator contract yet — listed here so the gap is explicit, not silently absent.

---

## Coverage

All ten Phase-1 objects plus the three reference types have complete, implementable contracts. Remaining caveats: TimeSpan's boundary vocabulary is provisional; NoteEvent's category boundary is unresolved; PitchIdentity's `token` representation is deferred to Deliverable D. Any future change to Domain Spec object shapes requires a re-sync pass here.
