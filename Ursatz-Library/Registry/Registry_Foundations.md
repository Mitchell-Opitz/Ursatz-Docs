# Registry — Foundations (L0–L4, Equality, Canonical IR)

**Last verified against repo state:** 2026-10-05 (Ursatz `090695c`, PR #97). Columns: Name | Category | Purpose |
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
FrameworkReference | Value | Opaque unowned reference to a Framework (L7, music-theory), by EntityID; lets L6 point at a Framework without depending on the music-theory module itself | music-core | EntityID | Interpretation, Chord, Key, ScaleDegree | 1 | Implemented (confirmed in effect, not merely proposed) | B | Yes | Value Contract | Invariant

**Note:** Interpretation-family types no longer depend on Framework directly. They depend on
`FrameworkReference` only (same reference-only pattern as VoiceReference/
RelationshipReference/ProvenanceReference). Framework resolution happens at L7+, never below.
No implementation may hold a live Framework reference at L6.

## Core / Value Objects

Entity | Category (base type) | Identifiable conceptual object | music-core | EntityID, SchemaVersion, Provenance | Work, Instrument, Voice, Motif, Framework, Rule | 1 | Implemented (base type, `src/core/entity.h`); Entity/Occurrence/Event boundary still contested upstream | B | Yes (BOUNDARY CONTESTED, vs Occurrence/Event) | Structural | Invariant
Value | Category (base type) | Immutable semantic content | music-core | — | Duration, MusicalTime, PitchClass, Interval, Tempo | 1 | Implemented (base type, `src/core/value.h`) | B | Yes | Structural | Property-Based
Occurrence | Category (base type) | Situated instance of a conceptual object | music-core | Structure/Motif/Theme (target) | MotifOccurrence, PhraseOccurrence, ThemeOccurrence | 1 | Implemented (base type, `src/core/occurrence.h`); composed by MotifOccurrence, ThemeOccurrence, PhraseOccurrence | B | Yes (BOUNDARY CONTESTED, vs Entity/Event) | Structural | Invariant, Context
Event | Category (base type) | Occurrence associated with a temporal domain | music-events | Occurrence, TimeAnchor | NoteEvent, RestEvent, ControlEvent | 1 | Implemented (base type, `src/events/event.h`, composes Occurrence + TemporalAnchor) | E | Yes (BOUNDARY CONTESTED, vs Occurrence/Entity) | Structural | Invariant
Container | Category (base type) | Groups/organizes objects | music-containers | — | Voice, Layer, Staff, Part, Measure, Phrase, Section, Work | 1 | Implemented (pure grouping marker, `src/containers/container.h`) | F | Yes | Structural | Invariant
Structure | Category (base type) | Higher-level semantic organization of material | music-structure | Event refs (non-duplicating) | Motif, Figure, Phrase, Theme, Section, Movement, Work | 5 | Implemented (base type with non-duplicating event refs, `src/structure/structure.h`) | F | Yes | Structural | Invariant
Assertion | Category (base type) | Claim made by an agent/process | music-interpretation | Agent | Observation, Hypothesis, Interpretation | 4 | Implemented (base type, `src/interpretation/assertion.h`) | I | Yes | Structural | Context, Theory
Observation | Category (base type) | Detected/measured info, no interpretation implied | music-interpretation | Source, Method, Uncertainty | Hypothesis, Candidate | 4 | Implemented (base type, `src/interpretation/observation.h`); no analyzer produces one yet | I | Yes | Structural | Context
Hypothesis | Category (base type) | Candidate explanation | music-interpretation | Observation, Evidence | Interpretation | 4 | Implemented (base type, `src/interpretation/hypothesis.h`); no analyzer produces one yet | I | Yes | Structural | Context
Interpretation | Category (base type) | Contextual claim about musical material | music-interpretation | Context, FrameworkReference, Evidence, Uncertainty, Provenance | Analysis results, Theory evaluation | 4 | Implemented. Sole output mechanism for every analyzer (Chord, Key, Voice Leading, Cadence, Motif, Phrase, Period, Sentence, Form, Thematic Return). Carries an optional FrameworkReference; Chord, Key, ScaleDegree and Texture require one | I | Yes | Structural | Context, Theory
Context | Category (base type) | Conditions under which an assertion is evaluated | music-context | Context (parent, composable) | Interpretation, Relationship, Framework evaluation | 4 | Implemented (base type, composable via parent reference, `src/context/context.h`); no analyzer builds one yet | H | Yes | Structural | Context
Transformation | Category (base type) | Pure domain operation producing new/derived objects | music-transform | Input type, Parameters, Provenance | Transposition, Inversion, Retrograde, etc. | 7 | Implemented (12 concrete transforms, including IntervallicManipulation added in PR #95); Fragmentation is NOT reusable for detection/matching | M | Yes | Behavioral Contract | Transformation, Golden

## Temporal Architecture

MusicalTime | Value | Exact symbolic temporal position (rational) | music-time | — | TimeSpan, NoteEvent, MeterMap, TempoMap | 1 | Implemented | C | Yes | Value Contract | Property-Based, Golden
Duration | Value | Amount of time (>=0, exact rational) | music-time | — | NoteEvent, RestEvent | 1 | Implemented | C | Yes | Value Contract | Property-Based, Invariant
TimeSpan | Value | Explicit start/end + boundary semantics | music-time | MusicalTime | Event, Structure occurrences | 1 | Implemented (boundary vocabulary remains provisional) | C | Yes | Value Contract | Invariant
TimeInstant | Value | Zero-duration temporal point | music-time | MusicalTime | TempoEvent, MeterChangeEvent | 1 | Implemented (`src/time/time_instant.h`) | C | Yes | Value Contract | Invariant
Meter | Value | Metrical organization (e.g., 4/4) | music-time | — | MeterMap | 2 | Implemented as `TimeSignature` (`src/time/time_signature.h`, PR #54): top number plus undotted bottom unit, no beat claim. Simple/compound reading lives in `frameworks/standard-meter`. Not yet produced by MIDIImporter | C | Yes | Value Contract | Unit
MeterMap | Entity | Associates meter with temporal regions | music-time | Meter, TimeSpan | MetricalPosition | 2 | Not Started | C | Yes | Structural | Unit
Tempo | Value | Tempo modeled independently of symbolic time | music-time | — | TempoMap | 2 | Implemented (partial) as `MetronomeMarking` (`src/time/`, exact-rational beat unit + bpm) and `TempoMarking` (`src/events/`, text and/or MetronomeMarking at a position), PR #52. Not read from MIDI | C | Yes | Value Contract | Unit
TempoMap | Entity | MusicalTime → PerformanceTime mapping | music-time | MusicalTime, PerformanceTime | Performance rendering | 2 | Not Started | C | Yes | Structural | Unit
NoteValue | Value | Written rhythmic value (breve to 1024th, up to 4 dots); converts one way to Duration | music-time | Duration | standard-rhythm, standard-meter | 2 | Implemented (PR #53); names and glyph conventions live in `frameworks/standard-rhythm` | C | Yes | Value Contract | Unit
Tuplet | Value | Ratio of notes played to notes replaced (3:2, 2:3, 5:4); scales a Duration exactly | music-time | Duration | standard-rhythm, standard-meter | 2 | Implemented (PR #56) | C | Yes | Value Contract | Unit
HistoricalPeriod | Value | Named span of calendar years, optionally open-ended; periods may overlap | music-time | — | western-periods | — | Implemented (PR #52). Historical, not musical, time; placement in `time/` accepted 2026-10-05 | — | No | Value Contract | Unit

*(TemporalAnchor is implemented (`src/time/temporal_anchor.h`). Notated/Performance/Audio time,
MetricalPosition and TemporalMapping are not started; see Registry_IO_Corpus_CrossCutting.md's
Performance/Recording section for the related I/O-facing types.)*

## Pitch Architecture

PitchIdentity | Value | Abstract pitch info, no tuning assumptions | music-pitch | — | PitchClass, PitchSpelling, PitchRealization | 1 | Implemented (equality resolved via strict token identity); deliberately opaque. Numeric work goes through PitchNumericValue (a separate, optional resolution under a TuningSystem), never by decoding PitchIdentity | D | Yes | Value Contract | Property-Based
PitchClass | Value | Equivalence under declared relation | music-pitch | PitchIdentity, equivalence model | Scale, Interval | 3 | Implemented (holds theory-relative equivalence via caller-supplied relation) | D | Yes | Value Contract | Property-Based
PitchSpelling | Value | Notation-oriented pitch identity (letter/accidental/octave) | music-pitch | PitchIdentity | NoteEvent (optional), Notation | 1 | Implemented (now persisted end-to-end, Ursatz PR #38) | D | Yes | Value Contract | Property-Based
PitchRealization | Value | Sounding/physical pitch (frequency, ratio, cents) | music-pitch | TuningSystem | PitchTrajectory, Performance | 3 | Implemented (type only). Wrapped by SoundingPitch; no analyzer consumes it | D | Yes | Value Contract | Unit
TuningSystem | Entity | Tuning definition (12-TET, JI, custom, etc.) | music-pitch | — | PitchRealization | 3 | Implemented (identity + name). Used by PitchNumericValue, SoundingPitch, Scale and MIDIImporter (a "MIDI note number" tuning). No analyzer computes real interval values from it yet | D | Yes | Structural | Unit
PitchNumericValue | Value | Exact-rational position of a pitch under one TuningSystem; supports add_interval, distance, reflect | music-pitch | PitchIdentity, TuningSystem | Transposition, Inversion, IntervallicManipulation, MIDIImporter | 3 | Implemented; one-way conversion to PitchIdentity, never back | D | Yes | Value Contract | Property-Based
WrittenPitch | Value | Notated (pre-transposition) pitch wrapping PitchSpelling | music-pitch | PitchSpelling | Notation | 3 | Implemented (thin wrapper) | D | Yes | Value Contract | Unit
SoundingPitch | Value | Sounding (post-transposition) pitch wrapping PitchRealization | music-pitch | PitchRealization | Performance | 3 | Implemented (thin wrapper) | D | Yes | Value Contract | Unit
Clef | Value | Clef sign anchored on a staff line, plus octave shift; neutral clef for unpitched staves | music-pitch | PitchSpelling | standard-notation | 2 | Implemented (PRs #49-50). Notation value accepted at L2 (2026-10-05 decision) | N | Yes | Value Contract | Unit
StaffPosition | Value | Notehead line/space step count from the bottom staff line | music-pitch | — | standard-notation | 2 | Implemented (PR #49). Notation value accepted at L2 | N | Yes | Value Contract | Unit
ClefResolution | Procedure | StaffPosition + Clef to letter/octave and back | music-pitch | Clef, StaffPosition | standard-notation | 2 | Implemented (PRs #49-50) | N | Yes | Behavioral Contract | Unit
Accidental | Value | Double flat to double sharp, maps to PitchSpelling's integer offset | music-pitch | PitchSpelling | standard-notation | 2 | Implemented (PR #51) | D | Yes | Value Contract | Unit
KeySignature | Value | Ordered letter + accidental entries, no theory claim | music-pitch | — | standard-notation, standard-tonality | 2 | Implemented (PR #57). Canonical key-signature value; MIDI key-signature scanner does not produce it yet | N | Yes | Value Contract | Unit
GenericInterval | Value | Written interval size counted in letters (1 = unison), no quality | music-pitch | PitchSpelling | standard-tonality | 3 | Implemented (PR #60). Not semantics/Interval (semitones between PitchIdentity tokens) | D | Yes | Value Contract | Unit
PitchTrajectory | Value | Continuous pitch over time (vibrato, glissando) | music-pitch | PitchRealization, time | Performance/Audio analysis | 3 | Not Started | D | Yes | Value Contract | Unit
Instrument | Entity | Range, transposition, clefs, technique, etc. | music-events / music-performance | — | InstrumentCapability, Voice, Generation constraints | 1+ | Implemented (id, name, range, transposition; clefs and technique deliberately omitted) | E | Partial (property list is illustrative) | Reference Concept | TBD

## Events

NoteEvent | Entity/Event | Symbolic musical occurrence | music-events | EntityID, TemporalAnchor, Duration, PitchIdentity, VoiceReference, ProvenanceReference | Interval, Sonority, PerformanceEvent | 1 | Implemented (category boundary below remains genuinely unresolved); Duration now confirmed tempo-independent whole-note-fraction units (PR #39) | E | Yes (BOUNDARY UNRESOLVED, Entity vs Event vs pure Occurrence/Value structure) | Structural (Unresolved) | TBD
RestEvent | Event | Explicit symbolic absence | music-events | TemporalAnchor, Duration | Analysis | 1 | Implemented (`src/events/rest_event.h`); MIDIImporter does not produce rests | E | Yes | Structural | Invariant
ControlEvent | Event | Non-note temporal change (pedal, CC, program change) | music-events | TemporalAnchor | Performance rendering | 1 | Implemented (opaque payload); MIDIImporter ignores control changes | E | Yes | Structural | Unit
Tie | Relationship/Attachment | Connects note events without merging | music-events | NoteEvent (x2) | Notation, Performance realization | 1+ | Implemented (references two NoteEvents by id) | E, G | Yes | Structural | Relationship
Slur | Attachment | Phrasing/legato/analytical grouping | music-events | NoteEvent range | Performance, Notation | 2 | Implemented (reference-only span, 2+ NoteEvents) | E | Yes | Structural | Relationship
Articulation | Attachment | Staccato, tenuto, accent, etc. | music-events | Event | Performance realization | 2 | Implemented (opaque kind string); meaning in `frameworks/standard-markings` | E | Yes | Structural | Unit
Dynamics | Attachment | p, mf, f, cresc./dim. | music-events | Event/TimeSpan | Performance realization | 2 | Implemented (opaque marking, attached to an Event or a TimeSpan); meaning in `frameworks/standard-markings` | E | Yes | Structural | Unit
TempoMarking | Event | Text and/or MetronomeMarking tempo indication at a position | music-events | TemporalAnchor, MetronomeMarking | standard-markings | 2 | Implemented (PR #52); not read from MIDI | E | Yes | Structural | Unit
StructuralMarker | Event | Fermata, repeat, ending, segno, etc. at a position | music-events | TemporalAnchor | standard-markings | 2 | Implemented (PR #52) | E | Yes | Structural | Unit
InstrumentCapability | Value | Range/transposition derived from an Instrument, separable from its identity | music-events | Instrument | Generation constraints | 1+ | Implemented | E | Partial | Reference Concept | TBD

## Containers

Voice | Container | Logical stream/simultaneity domain | music-containers | — | NoteEvent refs, Staff | 1 | Implemented (assignment now real, greedy interval-graph coloring by timing, scoped per track for multi-track single-channel MIDI; PRs #43, #46) | F | Yes | Structural | Invariant
Layer | Container | Notation-layer grouping | music-containers | — | Staff | 2 | Implemented (minimal `{id, grouping}`) | F | No (mentioned, no dedicated schema) | Reference Concept | TBD
Staff | Container | Notation staff grouping | music-containers | Voice(s), Layer(s) | Part, Score Model | 2 | Implemented (`{id, voice_ids, layer_ids}`) | F | Yes | Structural | Invariant
Part | Container | Instrument/performer grouping | music-containers | Staff(s) | Score Model | 2 | Implemented (`{id, staff_ids}`) | F | Yes | Structural | Invariant
Measure | Container | Metrical grouping | music-containers | MeterMap | Score Model | 2 | Implemented (`{id, meter_map_region_ref}`; MeterMap does not exist, so the ref is a bare EntityID) | F | No (example container) | Reference Concept | TBD
Phrase (container use) | Container | Mid-level grouping | music-containers | Event refs | Structure/Section | 5 | Implemented (partial); real, minimal L4 container, used only as a transient id-minting source by Phrase Boundary Detection and Form Structure Detection, not built out to full container semantics | F | No (example container) | Reference Concept | TBD
Section | Container | High-level grouping | music-containers | Phrase refs | Movement/Work | 5 | Implemented (partial); Form Structure Detection uses structural (PAC/IAC-only) cadences to bound sections | F | No (example container) | Reference Concept | TBD
Movement | Container | Work subdivision | music-containers | Section refs | Work | 5 | Implemented (`{id, section_ids}`) | F | No (example container) | Reference Concept | TBD
Work | Entity/Container | Top-level musical work | music-containers | Movement refs | Edition, Arrangement, Performance, Recording | 5 | Implemented (`{entity, movement_ids}`) | F | Yes | Structural | Invariant
Idea | Container | Smallest unit of a segmentation analysis: a short group of Event refs | music-containers | Event refs | Subphrase, standard-form | 5 | Implemented (PR #95). Not Motif | F | No | Structural | Invariant
Subphrase | Container | Group of Event refs between Idea and Phrase; need not end with a cadence | music-containers | Event refs | Phrase, standard-form | 5 | Implemented (PR #96) | F | No | Structural | Invariant

## Equality / Canonicalization

IdentityEquality | Concept | Same entity/version | music-core | EntityID, Version | Equality checks | 1 | Implemented (`*_identity_equals` per type) | A | Yes | Invariant | Invariant
ValueEquality | Concept | Semantic value equality | music-core | Value objects | Equality checks | 1 | Implemented (`*_equals_by_value` / value-equality per Value type) | A | Yes | Invariant | Invariant
StructuralEquality | Concept | Equivalent structure under rules | music-core | Structure rules | Equality checks | 1 | Implemented (`structural_equals`, `src/core/equality.h`) | A | Yes | Invariant | Invariant
SemanticEquality | Concept | Equivalent despite differing representation | music-core | Context/Tuning | Canonicalization | 1 | Implemented (contract only: `SemanticEqualsFn`; no concrete relation supplied) | A | Yes | Invariant | Invariant
CanonicalEquality | Concept | Same canonical normal form | music-core | Normalization Policy (versioned) | Serialization, Query | 1 | Implemented (contract: `canonical_equals` with a caller-supplied, versioned `CanonicalizeFn`) | A, O | Yes | Invariant | Serialization

## Canonical IR / Serialization / Persistence

Canonical Musical Model / IR | Concept | Theory-neutral representation of musical content | music-core | Event, Structure, Relationship, Context, Provenance | Everything downstream | 1 | Not Started (as a formal module; the emergent graph itself is real, see `Canonical_IR_Specification.md`'s Open Issue on this row's literal dependency wording) | N | Yes | Structural | Serialization, Invariant
Serialization Format(s) (JSON, protobuf, CBOR, XML, binary) | Adapter | Canonical object → external syntax | music-io | Canonical IR | Persistence, I/O | 1+ | Not Started | O | Yes (prohibition normative; specific formats are examples) | Behavioral Contract | Serialization
Canonical Serialization Spec | Concept | Schema version, stable IDs, deterministic ordering, exact rationals | music-io | Serialization | Migration | 1 | Not Started | O | Yes | Structural | Serialization
EventRepository | Interface | Repository for Event objects | music-persistence | Canonical IR | Application layer | 1 | Implemented (minimal/personal-scale) | P | Yes | Interface Contract | Unit
EntityRepository | Interface | Repository for Entity objects | music-persistence | Canonical IR | Application layer | 1 | Not Started | P | Yes | Interface Contract | Unit
RelationshipRepository | Interface | Repository for Relationship objects | music-persistence | Relationship | Application layer | 6 | Not Started in Ursatz proper. Ursatz-GUI added an application-layer stopgap SQLite table pending this; see `Dependency_Layer_Specification.md` | P | Yes | Interface Contract | Unit
StructureRepository | Interface | Repository for Structure objects | music-persistence | Structure | Application layer | 5 | Not Started (same caveat as RelationshipRepository, for Motif/Cadence/Theme/PhraseOccurrence L5 shells) | P | Yes | Interface Contract | Unit
InterpretationRepository | Interface | Repository for Interpretation objects | music-persistence | Interpretation | Application layer | 4 | Implemented (minimal/personal-scale) | P | Yes | Interface Contract | Unit
ProvenanceRepository | Interface | Repository for Provenance objects | music-persistence | Provenance | Application layer | 6 | Not Started | P | Yes | Interface Contract | Unit
Cache | Concept | Derived-value cache keyed by semantic dependencies | music-persistence | object_id, version, analyzer/framework version, context/config hash | Query performance | Post-Phase 1 | Not Started | P | Yes (key composition explicit) | Behavioral Contract | Performance
