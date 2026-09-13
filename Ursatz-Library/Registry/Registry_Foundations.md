# Registry — Foundations (L0–L4, Equality, Canonical IR)

**Last verified against repo state:** 2026-09-13. Columns: Name | Category | Purpose |
Module | Depends On | Depended On By | Phase | Status | Spec Ref | Normative? | Contract
Type | Test Spec Ref. See `Registry_Index.md` for the full-Registry split and standing
verification warning.

## Identity / Versioning

EntityID | Value | Stable identity for persistent objects | music-identity | — | Entity, all persistent objects | 1 | Implemented | A | Yes | Value Contract | Invariant, Serialization
Version | Value | Distinguish revisions from identity | music-identity | EntityID | Entity, VersionModel | 1 | Implemented | A | Yes | Value Contract | Invariant
VersionModel | Concept | Defines identity vs revision vs derivation semantics | music-identity | EntityID, Version | Entity, Provenance | 1 | Implemented | A | Yes | Structural | Invariant
SchemaVersion | Value | Tags schema of serialized/persisted objects | music-identity | — | Serialization, Persistence, Migration | 1 | Implemented | A, O | Yes | Value Contract | Serialization, Migration

## Core Reference Types

VoiceReference | Value | Opaque unowned reference to a Voice (L4), by EntityID | music-core | EntityID | NoteEvent | 1 | Implemented | B | Yes | Value Contract | Invariant
RelationshipReference | Value | Opaque unowned reference to a Relationship (L6), by EntityID | music-core | EntityID | Period Detection, Sentence Detection, Form Structure Detection (real consumers, confirmed 2026-09-13) | 1 | Implemented | B | Yes | Value Contract | Invariant
ProvenanceReference | Value | Opaque unowned reference to Provenance (L6/music-provenance), by EntityID, three-state presence (absent/pending/present) | music-core | EntityID | NoteEvent, Relationship, Interpretation, and other L3-L8 objects requiring provenance | 1 | Implemented | B | Yes | Value Contract | Invariant
FrameworkReference | Value | Opaque unowned reference to a Framework (L7, music-theory), by EntityID — lets L6 point at a Framework without depending on the music-theory module itself | music-core | EntityID | Interpretation, Chord, Key, ScaleDegree | 1 | Implemented — confirmed in effect, not merely proposed | B | Yes | Value Contract | Invariant

**Note:** Interpretation-family types no longer depend on Framework directly. They depend on
`FrameworkReference` only (same reference-only pattern as VoiceReference/
RelationshipReference/ProvenanceReference). Framework resolution happens at L7+, never below.
No implementation may hold a live Framework reference at L6.

## Core / Value Objects

Entity | Category (base type) | Identifiable conceptual object | music-core | EntityID, SchemaVersion, Provenance | Work, Instrument, Voice, Motif, Framework, Rule | 1 | Not Started | B | Yes — BOUNDARY CONTESTED (vs Occurrence/Event) | Structural | Invariant
Value | Category (base type) | Immutable semantic content | music-core | — | Duration, MusicalTime, PitchClass, Interval, Tempo | 1 | Not Started | B | Yes | Structural | Property-Based
Occurrence | Category (base type) | Situated instance of a conceptual object | music-core | Structure/Motif/Theme (target) | MotifOccurrence, PhraseOccurrence, ThemeOccurrence | 1 | Not Started | B | Yes — BOUNDARY CONTESTED (vs Entity/Event) | Structural | Invariant, Context
Event | Category (base type) | Occurrence associated with a temporal domain | music-events | Occurrence, TimeAnchor | NoteEvent, RestEvent, ControlEvent | 1 | Not Started | E | Yes — BOUNDARY CONTESTED (vs Occurrence/Entity) | Structural | Invariant
Container | Category (base type) | Groups/organizes objects | music-containers | — | Voice, Layer, Staff, Part, Measure, Phrase, Section, Work | 1 | Not Started | F | Yes | Structural | Invariant
Structure | Category (base type) | Higher-level semantic organization of material | music-structure | Event refs (non-duplicating) | Motif, Figure, Phrase, Theme, Section, Movement, Work | 5 | Not Started | F | Yes | Structural | Invariant
Assertion | Category (base type) | Claim made by an agent/process | music-interpretation | Agent | Observation, Hypothesis, Interpretation | 4 | Not Started | I | Yes | Structural | Context, Theory
Observation | Category (base type) | Detected/measured info, no interpretation implied | music-interpretation | Source, Method, Uncertainty | Hypothesis, Candidate | 4 | Not Started | I | Yes | Structural | Context
Hypothesis | Category (base type) | Candidate explanation | music-interpretation | Observation, Evidence | Interpretation | 4 | Not Started | I | Yes | Structural | Context
Interpretation | Category (base type) | Contextual claim about musical material | music-interpretation | Context, FrameworkReference, Evidence, Uncertainty, Provenance | Analysis results, Theory evaluation | 4 | Implemented (partial) — thin-wrapper form confirmed in code (claim persisted opaquely) and used as the sole output mechanism for every analyzer built so far (Chord, Key, Voice Leading, Cadence, Motif, Phrase, Period, Sentence, Form); full {target, claim, framework, context, evidence, uncertainty, author, method, status, provenance} shape per TDD not fully confirmed field-by-field | I | Yes | Structural | Context, Theory
Context | Category (base type) | Conditions under which an assertion is evaluated | music-context | Context (parent, composable) | Interpretation, Relationship, Framework evaluation | 4 | Not Started | H | Yes | Structural | Context
Transformation | Category (base type) | Pure domain operation producing new/derived objects | music-transform | Input type, Parameters, Provenance | Transposition, Inversion, Retrograde, etc. | 7 | Implemented — all 11 concrete transforms implemented, tested, valgrind-clean; confirmed Fragmentation is NOT reusable for detection/matching (only clones caller-supplied event ids, no comparison logic — Sentence Detection needed new comparison logic instead) | M | Yes | Behavioral Contract | Transformation, Golden

## Temporal Architecture

MusicalTime | Value | Exact symbolic temporal position (rational) | music-time | — | TimeSpan, NoteEvent, MeterMap, TempoMap | 1 | Implemented | C | Yes | Value Contract | Property-Based, Golden
Duration | Value | Amount of time (>=0, exact rational) | music-time | — | NoteEvent, RestEvent | 1 | Implemented | C | Yes | Value Contract | Property-Based, Invariant
TimeSpan | Value | Explicit start/end + boundary semantics | music-time | MusicalTime | Event, Structure occurrences | 1 | Implemented — boundary vocabulary remains provisional | C | Yes | Value Contract | Invariant
TimeInstant | Value | Zero-duration temporal point | music-time | MusicalTime | TempoEvent, MeterChangeEvent | 1 | Not Started | C | Yes | Value Contract | Invariant
Meter | Value | Metrical organization (e.g., 4/4) | music-time | — | MeterMap | 2 | Not Started | C | Yes | Value Contract | Unit
MeterMap | Entity | Associates meter with temporal regions | music-time | Meter, TimeSpan | MetricalPosition | 2 | Not Started | C | Yes | Structural | Unit
Tempo | Value | Tempo modeled independently of symbolic time | music-time | — | TempoMap | 2 | Not Started | C | Yes | Value Contract | Unit
TempoMap | Entity | MusicalTime → PerformanceTime mapping | music-time | MusicalTime, PerformanceTime | Performance rendering | 2 | Not Started | C | Yes | Structural | Unit

*(Notated/Performance/Audio time, TemporalAnchor, MetricalPosition, TemporalMapping: all Not
Started — see Registry_IO_Corpus_CrossCutting.md's Performance/Recording section for the
related I/O-facing types.)*

## Pitch Architecture

PitchIdentity | Value | Abstract pitch info, no tuning assumptions | music-pitch | — | PitchClass, PitchSpelling, PitchRealization | 1 | Implemented — equality resolved (strict token identity); deliberately opaque, no numeric/tuning semantics — real interval/direction computation from PitchIdentity is not possible without TuningSystem/PitchRealization | D | Yes | Value Contract | Property-Based
PitchClass | Value | Equivalence under declared relation | music-pitch | PitchIdentity, equivalence model | Scale, Interval | 3 | Implemented — holds theory-relative equivalence via caller-supplied relation | D | Yes | Value Contract | Property-Based
PitchSpelling | Value | Notation-oriented pitch identity (letter/accidental/octave) | music-pitch | PitchIdentity | NoteEvent (optional), Notation | 1 | Implemented — now persisted end-to-end (Ursatz PR #38) | D | Yes | Value Contract | Property-Based
PitchRealization | Value | Sounding/physical pitch (frequency, ratio, cents) | music-pitch | TuningSystem | PitchTrajectory, Performance | 3 | Implemented (type only) — verified 2026-09-13: exists in `src/pitch/`, but no analyzer consumes it. Remains a concrete blocker on Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif Detection computing real values — see `Known_Gaps.md` | D | Yes | Value Contract | Unit
TuningSystem | Entity | Tuning definition (12-TET, JI, custom, etc.) | music-pitch | — | PitchRealization | 3 | Implemented (type only) — same consumption gap as PitchRealization above | D | Yes | Structural | Unit
PitchTrajectory | Value | Continuous pitch over time (vibrato, glissando) | music-pitch | PitchRealization, time | Performance/Audio analysis | 3 | Not Started | D | Yes | Value Contract | Unit
Instrument | Entity | Range, transposition, clefs, technique, etc. | music-events / music-performance | — | InstrumentCapability, Voice, Generation constraints | 1+ | Not Started | E | Partial (property list is illustrative) | Reference Concept | TBD

## Events

NoteEvent | Entity/Event | Symbolic musical occurrence | music-events | EntityID, TemporalAnchor, Duration, PitchIdentity, VoiceReference, ProvenanceReference | Interval, Sonority, PerformanceEvent | 1 | Implemented — category boundary below remains genuinely unresolved; Duration now confirmed tempo-independent whole-note-fraction units (PR #39) | E | Yes — BOUNDARY UNRESOLVED (Entity vs Event vs pure Occurrence/Value structure) | Structural (Unresolved) | TBD
RestEvent | Event | Explicit symbolic absence | music-events | TemporalAnchor, Duration | Analysis | 1 | Not Started | E | Yes | Structural | Invariant
ControlEvent | Event | Non-note temporal change (pedal, CC, program change) | music-events | TemporalAnchor | Performance rendering | 1 | Not Started | E | Yes | Structural | Unit
Tie | Relationship/Attachment | Connects note events without merging | music-events | NoteEvent (x2) | Notation, Performance realization | 1+ | Not Started | E, G | Yes | Structural | Relationship
Slur | Attachment | Phrasing/legato/analytical grouping | music-events | NoteEvent range | Performance, Notation | 2 | Not Started | E | Yes | Structural | Relationship
Articulation | Attachment | Staccato, tenuto, accent, etc. | music-events | Event | Performance realization | 2 | Not Started | E | Yes | Structural | Unit
Dynamics | Attachment | p, mf, f, cresc./dim. | music-events | Event/TimeSpan | Performance realization | 2 | Not Started | E | Yes | Structural | Unit

## Containers

Voice | Container | Logical stream/simultaneity domain | music-containers | — | NoteEvent refs, Staff | 1 | Implemented — assignment now real: greedy interval-graph coloring by timing, scoped per track for multi-track single-channel MIDI (PRs #43, #46) | F | Yes | Structural | Invariant
Layer | Container | Notation-layer grouping | music-containers | — | Staff | 2 | Not Started | F | No (mentioned, no dedicated schema) | Reference Concept | TBD
Staff | Container | Notation staff grouping | music-containers | Voice(s), Layer(s) | Part, Score Model | 2 | Not Started | F | Yes | Structural | Invariant
Part | Container | Instrument/performer grouping | music-containers | Staff(s) | Score Model | 2 | Not Started | F | Yes | Structural | Invariant
Measure | Container | Metrical grouping | music-containers | MeterMap | Score Model | 2 | Not Started | F | No (example container) | Reference Concept | TBD
Phrase (container use) | Container | Mid-level grouping | music-containers | Event refs | Structure/Section | 5 | Implemented (partial) — real, minimal L4 container; used only as a transient id-minting source by Phrase Boundary Detection and Form Structure Detection, not built out to full container semantics | F | No (example container) | Reference Concept | TBD
Section | Container | High-level grouping | music-containers | Phrase refs | Movement/Work | 5 | Implemented (partial) — Form Structure Detection uses structural (PAC/IAC-only) cadences to bound sections | F | No (example container) | Reference Concept | TBD
Movement | Container | Work subdivision | music-containers | Section refs | Work | 5 | Not Started | F | No (example container) | Reference Concept | TBD
Work | Entity/Container | Top-level musical work | music-containers | Movement refs | Edition, Arrangement, Performance, Recording | 5 | Not Started | F | Yes | Structural | Invariant

## Equality / Canonicalization

IdentityEquality | Concept | Same entity/version | music-core | EntityID, Version | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
ValueEquality | Concept | Semantic value equality | music-core | Value objects | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
StructuralEquality | Concept | Equivalent structure under rules | music-core | Structure rules | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
SemanticEquality | Concept | Equivalent despite differing representation | music-core | Context/Tuning | Canonicalization | 1 | Not Started | A | Yes | Invariant | Invariant
CanonicalEquality | Concept | Same canonical normal form | music-core | Normalization Policy (versioned) | Serialization, Query | 1 | Not Started | A, O | Yes | Invariant | Serialization

## Canonical IR / Serialization / Persistence

Canonical Musical Model / IR | Concept | Theory-neutral representation of musical content | music-core | Event, Structure, Relationship, Context, Provenance | Everything downstream | 1 | Not Started (as a formal module — the emergent graph itself is real; see `Canonical_IR_Specification.md`'s Open Issue on this row's literal dependency wording) | N | Yes | Structural | Serialization, Invariant
Serialization Format(s) (JSON, protobuf, CBOR, XML, binary) | Adapter | Canonical object → external syntax | music-io | Canonical IR | Persistence, I/O | 1+ | Not Started | O | Yes (prohibition normative; specific formats are examples) | Behavioral Contract | Serialization
Canonical Serialization Spec | Concept | Schema version, stable IDs, deterministic ordering, exact rationals | music-io | Serialization | Migration | 1 | Not Started | O | Yes | Structural | Serialization
EventRepository | Interface | Repository for Event objects | music-persistence | Canonical IR | Application layer | 1 | Implemented (minimal/personal-scale) | P | Yes | Interface Contract | Unit
EntityRepository | Interface | Repository for Entity objects | music-persistence | Canonical IR | Application layer | 1 | Not Started | P | Yes | Interface Contract | Unit
RelationshipRepository | Interface | Repository for Relationship objects | music-persistence | Relationship | Application layer | 6 | Not Started in Ursatz proper — Ursatz-GUI added an application-layer stopgap SQLite table pending this; see `Dependency_Layer_Specification.md` | P | Yes | Interface Contract | Unit
StructureRepository | Interface | Repository for Structure objects | music-persistence | Structure | Application layer | 5 | Not Started — same caveat as RelationshipRepository, for Motif/Cadence/Theme/PhraseOccurrence L5 shells | P | Yes | Interface Contract | Unit
InterpretationRepository | Interface | Repository for Interpretation objects | music-persistence | Interpretation | Application layer | 4 | Implemented (minimal/personal-scale) | P | Yes | Interface Contract | Unit
ProvenanceRepository | Interface | Repository for Provenance objects | music-persistence | Provenance | Application layer | 6 | Not Started | P | Yes | Interface Contract | Unit
Cache | Concept | Derived-value cache keyed by semantic dependencies | music-persistence | object_id, version, analyzer/framework version, context/config hash | Query performance | Post-Phase 1 | Not Started | P | Yes (key composition explicit) | Behavioral Contract | Performance
