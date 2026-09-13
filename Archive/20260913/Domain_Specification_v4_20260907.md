# DOMAIN ONTOLOGY SPECIFICATION (Deliverable B) v4

Source: TDD v4 §233/§260, System Registry v3, Dependency & Layer Specification v3
Supersedes: v3. Condensed — resolved deliberation and superseded-version diffing removed; normative content and open issues preserved.

## Status

Phase-1 ontology objects below are implemented and Registry-registered, including `VoiceReference`, `RelationshipReference`, and `ProvenanceReference` (Registry v3 closed their registration gate). Two items remain genuinely open — see §Open Issues.

---

## Normative rule: reference-only dependency vs. module dependency

A lower-layer object may carry an opaque reference to an object conceptually belonging to a higher layer **only when**:
1. the reference type itself is defined in the lower layer (or an allowed cross-cutting module), not the higher layer's module;
2. holding/serializing that reference does not require the lower-layer module to import the higher-layer module;
3. resolving the reference into the actual object happens only in code at or above the higher layer.

This authorizes exactly one pattern: an opaque, EntityID-shaped pointer, defined low, resolved high. It does not authorize embedding any semantics, fields, or behavior of the higher-layer object into the lower layer. This rule is why `VoiceReference` and `RelationshipReference` exist (§ below) rather than NoteEvent holding a direct `Voice`/`Relationship` field.

---

## Object catalog

### EntityID — L0 (music-identity)
Opaque token naming one conceptual object across its entire lifetime, including all revisions. Answers "which thing," never "where" or "what version." Value equality. Invariants: non-empty; MUST NOT encode positional information (Law 16); stable for the entity's lifetime. Serializes as a stable string/opaque scalar, never a row number or array index. Assigned once at creation, never reassigned or reused after tombstoning.

### Duration — L2 (music-time)
Exact rational amount of symbolic time, independent of when it starts. Fields: `numerator: Integer`, `denominator: PositiveInteger`. Value equality on the *reduced* rational (3/4 == 6/8). Invariants: duration ≥ 0 (Law 19); denominator > 0; MUST NOT use floating point.

### MusicalTime — L2 (music-time)
Exact rational *position* ("when"), structurally identical to Duration but semantically distinct — never used interchangeably. Same field shape, same no-float invariant. No inherent lower bound (negative time for pickups/anacrusis is a Deliverable-C modeling decision, not fixed here). Distinct from NotatedTime/PerformanceTime/AudioTime — no implicit conversion at the kernel level.

### TimeSpan — L2 (music-time)
Explicit temporal region: `start: MusicalTime`, `end: MusicalTime`, `boundary: <working vocabulary, see below>`. Value equality includes boundary — two spans with equal start/end but different boundary are NOT equal. Invariant: `end >= start`; boundary MUST be explicit, never implicit/defaulted on deserialization.
**Boundary vocabulary is provisional, not frozen.** `closed / open / left_closed / right_closed` is the working set (TDD's own illustrative list); Deliverable C owns the authoritative version and may extend/rename it without breaking this document's intent, provided explicitness and value-equality inclusion are preserved.

### PitchIdentity — L2 (music-pitch)
Root of the pitch dimension hierarchy (→ PitchClass, PitchSpelling, PitchRealization). Names "a pitch, abstractly" — asserts nothing about tuning, pitch-class cardinality, or frequency (Law 18). Field: `token: opaque internal representation` — explicitly NOT an EntityID; carries no lifecycle/versioning semantics, exists only to make PitchIdentity comparable. Value object, no identity.
**Equality — RESOLVED:** strict identity on the opaque token. Coarser equivalence (octave, enharmonic) is answered by `PitchClass` (music-pitch, implemented), which takes a caller-supplied equivalence relation — not by PitchIdentity itself. This is final, not a placeholder.
Invariants: MUST NOT assume 12 pitch classes, equal temperament, A4=440Hz, or integer MIDI numbers; MUST NOT carry EntityID/entity-lifecycle semantics.

### PitchSpelling — L2 (music-pitch)
Notation-oriented pitch identity (letter/accidental/octave), independent of sounding realization — preserves the C#/Db distinction even when PitchRealization coincides (Law 18). Fields: `letter: enum{A..G}`, `accidental: Integer|enum` (exact model deferred to Deliverable D), `octave: Integer`. Value equality on the (letter, accidental, octave) triple — enharmonic equivalents are NOT value-equal. Depends on PitchIdentity.

### NoteEvent — L3 (music-events)
**Category placement (Entity vs. Event vs. pure Occurrence/Value) remains open — see Open Issues.** Documented here under the provisional Event-category treatment.

The canonical, symbolic representation of "a note occurring." Explicitly NOT a notation symbol, MIDI message, performed sound, scale degree, or chord member (Laws 3/4/5).

Fields:
```
id: EntityID
temporal_anchor: TemporalAnchor (wraps MusicalTime)
duration: Duration
pitch: PitchIdentity
optional_pitch_spelling: PitchSpelling
optional_voice: VoiceReference        -- opaque L1 reference, not a Voice-typed field
provenance: ProvenanceReference       -- present/absent/omitted-is-invalid, see below
```
`music-events` (L3) never imports `music-containers` (L4) — the Voice relationship is reference-only, resolved by code at L4+.

Identity equality is primary (same EntityID+Version = same event); value/structural equality of two different identities does not imply sameness. Invariants: id stable and position-independent (Law 16); duration ≥ 0; pitch must reference a valid PitchIdentity; provenance must be present or explicitly absent, never silently omitted; a present provenance must resolve, never dangle; no instrument-transposition baked into pitch (Law 18).

Lifecycle: creation → revision → derivation → supersession → deletion/tombstoning. Transformations create new NoteEvent identities with provenance links back to source events (Law 20) rather than mutating in place.

### Voice — L4 (music-containers)
A logical stream/simultaneity grouping of NoteEvents — semantically inert with respect to orchestration/notation (TDD §53): does not imply an instrument, staff, part, or harmonic/melodic role. Field: `id: EntityID`; membership represented via event references, not an owned/mutable list. May appear on multiple staves; a Staff may contain multiple voices. Distinct from Layer (notation-layer grouping) and Part (instrument/performer grouping). May depend on music-events (L3) — permitted, L4→L3.

### VoiceReference — L1 (music-core)
Resolves the NoteEvent→Voice layer contradiction under the reference-only rule above. Opaque `EntityID`-shaped pointer to a Voice (L4), defined at L1 so `music-events` never imports `music-containers`. Value equality on the referenced EntityID. Resolution into an actual Voice happens only at L4+.

### RelationshipReference — L1 (music-core)
Lets Phase-1 objects (NoteEvent, Voice) participate in the first-class Relationship model (L6, TDD §75) without importing the full Relationship machinery. Field: `relationship_id: EntityID reference`, resolved only at L6+. Deliberately located at L1, not inside `music-relationship`, so lower layers can hold it legally. MUST NOT embed predicate/scope/context/evidence — those stay at L6.

### ProvenanceReference — cross-cutting (L0–L1 ceiling), music-provenance
Lets Phase-1 objects carry lineage pointers (Law 10) without inlining the full Provenance/Activity/Agent/Derivation model. Any L1+ layer may import `music-provenance` directly (cross-cutting, no relocation needed).

**Three-state presence model:**
```
present  → state=present; provenance_id populated, MUST resolve to a real record.
absent   → state=absent; explicit, first-class assertion that no provenance applies —
           reserved for genuinely provenance-free objects (hand-authored fixtures, seed data),
           never for anything produced by an Activity (import/transform/generate/analyze).
omitted  → field not represented at all. INVALID wherever the schema requires provenance.
           MUST NOT be treated as equivalent to "absent."
```
Value equality on `(state, provenance_id)` — two `absent` references are equal regardless of `absence_reason` (a flagged, revisable modeling choice, not settled doctrine). Invariant: `state` must always be explicit; a present-state dangling reference is a violation, not a warning.

---

## Registry synchronization

`VoiceReference`, `RelationshipReference`, and `ProvenanceReference` are Registry-registered as of Registry v3 (previously proposed-only in v0.2/v0.3 of this document). NoteEvent's Registry row correctly reflects Voice/Relationship as reference-only dependencies, not module dependencies:
```
Module deps: EntityID, TemporalAnchor, Duration, PitchIdentity, VoiceReference (L1),
             RelationshipReference (L1), ProvenanceReference (cross-cutting)
Reference-only: Voice (L4, via VoiceReference), Relationship (L6, via RelationshipReference)
```

---

## Open Issues

### Entity vs. Occurrence vs. Event category boundary
Still unresolved. Affects NoteEvent's category placement and, by extension, Structure/Occurrence relationships. No implementation or architecture test should assume this is settled.

### TimeSpan boundary vocabulary
Still provisional. `closed/open/left_closed/right_closed` is the working set; Deliverable C owns the authoritative version.

### ProvenanceReference equality over `absence_reason`
Excluding `absence_reason` from value equality is a reasonable default, explicitly flagged as revisable by Deliverable K, not a settled conclusion.

---

## Resolved (no longer open)

- **PitchIdentity token representation/equivalence** — resolved v0.3: strict token identity; coarser equivalence handled by PitchClass.
- **Interpretation-family → Framework ordering** — resolved via `FrameworkReference` (Registry v3); closes Dependency Spec §7.1.
- **VoiceReference / RelationshipReference / ProvenanceReference Registry authorization** — resolved, Registry v3.

---

## Coverage

All ten Phase-1 objects (EntityID, Duration, MusicalTime, TimeSpan, PitchIdentity, PitchSpelling, NoteEvent, Voice, VoiceReference-family) are specified and implemented. Next pass: base categories (Entity, Value, Occurrence), then Phase 2 objects (Meter, MeterMap, Measure, Clef, Staff, Part, NotationPosition, Tempo, TempoMap, Dynamics, Articulation).
