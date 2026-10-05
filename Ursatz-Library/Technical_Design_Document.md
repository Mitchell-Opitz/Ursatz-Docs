# Technical Design Document — Ursatz Kernel

**Last verified against repo state:** 2026-10-05 (Ursatz `090695c`, PR #97). This document is the source for kernel
scope/rationale not already covered by `Overview/Architecture_Principles.md` (25 Laws, layer
model) or the dedicated specs (Domain, API Contract, Canonical IR, Dependency & Layer, Test).
Read those first; this fills in what's left.

**Primary Objective:** a theory-neutral computational foundation for representing,
observing, interpreting, analyzing, transforming, generating, performing, rendering,
importing, exporting, querying, and studying musical material, without any theory,
notation system, storage technology, or generation technology defining the underlying
musical domain.

## Domain boundaries

World/Source, Digital Artifact, Musical, Representation, Observation, Interpretation,
Theory/Knowledge, Transformation, Generation, Performance, Query, Corpus, Persistence. These
are architectural boundaries, not merely organizational ones; a plugin/consumer may not
collapse them for convenience.

## Full phase roadmap (original design intent — see `Status.md` for actual current state)

| Phase | Scope | Design-time status |
|---|---|---|
| 1 | Kernel: EntityID, Duration, MusicalTime, TimeSpan, PitchIdentity, PitchSpelling, NoteEvent, Voice, RelationshipReference, ProvenanceReference | Complete |
| 2 | Temporal/Score: Meter, MeterMap, Measure, Clef, Staff, Part, NotationPosition, Tempo, TempoMap, Dynamics, Articulation | Partial. Value types built: TimeSignature (Meter), Clef, StaffPosition, KeySignature, NoteValue, Tuplet, MetronomeMarking, TempoMarking; Measure, Staff, Part, Dynamics, Articulation built. MeterMap, TempoMap not started; MIDI import reads neither meter nor tempo |
| 3 | Pitch/Semantics: PitchClass, PitchRealization, Tuning, Interval, Sonority, Scale, ScaleDegree, Contour, RhythmicPattern | Implemented (plus PitchNumericValue, GenericInterval). PitchTrajectory not started |
| 4 | Context/Interpretation: Context, Observation, Candidate, Hypothesis, Interpretation, Evidence, Uncertainty, Framework | Implemented as base types. Uncertainty has 3 of 8 kinds. Analyzers produce Interpretation + Evidence only |
| 5 | Structures: Motif, Occurrence, Figure, Phrase, Cadence, Theme, Section, Movement, Work | Implemented as L5 shells |
| 6 | Relationships/Provenance: Relationship, Predicate, Evidence, ProvenanceActivity/Agent, Derivation, AuditEvent | Relationship implemented and in real use; Provenance model beyond the reference type not started |
| 7 | Transformation: concrete transforms | Implemented (12, including IntervallicManipulation) |
| 8 | Analysis: all analyzers | Implemented (see `Status.md`) |
| 9 | Theory Framework: Framework, Rule, Constraint, Preference, Heuristic, Tendency, Procedure, StatisticalModel | Implemented as base types except Procedure. Constraint/Preference evaluation deliberately stubbed; Tendency/StatisticalModel shape only |
| 10 | Species Counterpoint | Implemented as a rule library (`frameworks/standard-counterpoint`, species 1-5) |
| 11 | Common-Practice Harmony (full plugin framework, distinct from `common-practice-minimal`) | Partial: rule libraries built (`standard-tonality`, `standard-harmony`); the harmonic analyzer that feeds them from NoteEvents is not |
| 12 | Generation | Base types implemented; Generator/IntentCompiler orchestration not started |
| 13 | Hierarchical Generation | Not started |
| 14 | I/O: MIDI, MusicXML, MEI, ABC | MIDI import + MusicXML export implemented; MEI/ABC not started |
| 15 | Corpus | Implemented (minimal) |
| 16 | OCR | Not started |
| 17 | Audio | Not started |
| 18 | Advanced Frameworks | Style-period frameworks built as rule libraries (16th-century, high-Baroque fugue, ground bass, Galant schemas, scale harmonization); the listed examples (jazz, Schenkerian, etc.) not started |

## Phase-0 deliverable inventory

| Deliverable | Owning doc |
|---|---|
| A. Identity and Version | Folded into Domain Specification |
| B. Domain Ontology | `Domain_Specification.md` |
| C. Temporal Semantics | Not yet written as a standalone doc (owns TimeSpan boundary vocabulary, still provisional) |
| D. Pitch and Tuning | Not yet written (owns PitchIdentity token representation) |
| E. Musical Event | Folded into Domain Specification |
| F. Container and Structure | Not yet written (owns Voice membership mechanics) |
| G. Relationship | Not yet written (Relationship itself is implemented; the spec describing it formally is not) |
| H. Context | Not yet written |
| I. Observation/Hypothesis/Interpretation | Not yet written |
| J. Evidence and Uncertainty | Not yet written |
| K. Provenance | Not yet written (owns Derivation record shape) |
| L. Theory/Rule/Constraint | Not yet written |
| M. Transformation | Not yet written |
| N. Canonical IR | `Canonical_IR_Specification.md` |
| O. Serialization | Not yet written |
| P. Persistence | Not yet written (minimal SQLite persistence implemented ahead of this spec) |
| Q. Query Semantics | Not yet written |
| R. Validation | Not yet written |
| S. Plugin/Capability | Not yet written |
| T. I/O and Loss | Not yet written (MIDIImporter + MusicXmlExport implemented ahead of this spec) |
| U. Testing | `Test_Specification.md` |
| (unlettered) Dependency & Layer direction | `Dependency_Layer_Specification.md` |

Several Phase-1 implementations (persistence, MIDI import, MusicXML export, Key
Estimation/Chord Identification) shipped ahead of their owning deliverable specs (P, T, L)
being formally written. This is a known, disclosed gap, not a silent one; the same
"disclosed scope cuts" principle in `Overview/Master_Roadmap.md` applies here.

## Architecture risks & mitigations (design-time, still the working list)

| Risk | Mitigation |
|---|---|
| Scope explosion | tiny kernel, phased development, vertical slices, extension boundaries |
| Ontology explosion | prefer relationships, derive values, use contextual interpretations |
| Theory leakage | dependency architecture tests, framework isolation |
| Interpretation becomes fact | observation/hypothesis/interpretation/evidence/uncertainty/provenance pipeline |
| Time model failure | exact rational symbolic time + explicit anchors + tempo mappings |
| Pitch model failure | identity + spelling + realization + tuning + trajectory, kept separate |
| Uncertainty collapse | unknown + candidate sets + probabilistic models + competing hypotheses |
| Plugin security | capabilities, sandboxing, resource limits (none of this built yet — Phase 9+) |

## Extension mechanism

New semantic concepts enter as framework-defined semantic objects, relationships,
interpretations, procedures, or annotations, not by modifying core classes. A new concept
enters the foundational kernel only if: (1) framework-neutral, (2) required by multiple
subsystems, (3) not reasonably representable as an extension, (4) semantically stable,
(5) introduces no theory leakage.

**Narrow exception (2026-10-05): vocabulary enums.** A plain enum that only names a theory
concept (no EntityID, no data, no framework claim) may live in `src/semantics/` when two or
more frameworks need the same name. See `Dependency_Layer_Specification.md`.
