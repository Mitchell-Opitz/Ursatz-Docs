# Registry — Query, Notation, I/O, Corpus, Cross-Cutting, Transformation Subtypes (L9 + cross-cutting)

**Last verified against repo state:** 2026-09-13. Columns: Name | Category | Purpose |
Module | Depends On | Depended On By | Phase | Status | Spec Ref | Normative? | Contract
Type | Test Spec Ref. See `Registry_Index.md` for the full-Registry split and standing
verification warning.

## Query

Query Interface (find_notes, find_intervals, find_occurrences, find_interpretations, find_relationships) | Interface | Domain queries replacing raw SQL | music-query | Repositories | Applications | Post kernel | Not Started | Q | Yes | Interface Contract | Unit
PatternMatching | Concept | Match over declared dimensions (interval, rhythm) | music-query | Interval/RhythmicPattern | Motif Detection, Search | 8 | Not Started — Motif Detection (implemented) deliberately does NOT depend on this; it does direct Contour/RhythmicPattern sequence comparison instead, to avoid standing up unbuilt L9 query infrastructure | Q | Yes | Behavioral Contract | Unit
SimilarityModel | Concept | Explicit-dimension similarity (pitch, contour, rhythm, transposition, etc.) | music-query | Interval, Contour, RhythmicPattern | SearchResult | 8 | Not Started — same scope-avoidance note as PatternMatching above | Q | Yes (no undeclared-similarity prohibition is explicit) | Behavioral Contract | Unit
SearchResult | Value | Target, match region, similarity, criteria, evidence, provenance | music-query | PatternMatching/SimilarityModel | Application UI | 8 | Not Started | Q | Yes | Structural | Unit
Views (Temporal, Pitch, Rhythmic, Voice, Harmonic, Motivic, Formal, Performance, Notation) | Concept | Derived, non-duplicating projections of canonical data | music-query | Canonical IR | Applications | Post kernel | Not Started | Q | Yes | Structural | Invariant

## Notation / Score

Score Model (Staff, Clef, KeySignature, TimeSignature, Measure, Beam, Stem, Barline, Tuplet, Layout, Page, System) | Concept group | Specialized representation layer, non-authoritative over canonical events | music-notation | Canonical Events, Containers | Notation rendering | 2 | Not Started | F, N | Yes (non-authority rule explicit; internal item list illustrative) | Structural | Invariant
Notation Document Model | Concept | Preserves engraving/layout/editorial info without canonical-event equivalent | music-notation | DigitalArtifact | MusicXML/MEI adapters | 2/14 | Not Started | N | Yes | Structural | RoundTrip

**Note:** Ursatz-GUI renders notation via a real MusicXML export (see I/O Adapters below)
consumed by a vendored OpenSheetMusicDisplay frontend — an application-level rendering
solution, not an implementation of this kernel module. The kernel-level Score Model/Notation
Document Model remain entirely unbuilt.

## World / Source / Digital Artifact

Source | Entity | External artifact/phenomenon with identity/provenance | (cross-cutting) | — | DigitalArtifact, Recording | 1 | Not Started | B | Yes | Structural | Provenance
DigitalArtifact | Entity | External digital object (PDF, MusicXML, MIDI, MEI, WAV, JSON, image, DB export) | music-io | Source | Import pipeline | 1 | Not Started | T | Yes | Structural | Invariant

## I/O Adapters

MIDIImporter | Adapter | MIDI → Observations/Candidates → Canonical | music-io | MIDI transport concepts | Canonical IR | 14 | Implemented (minimal) — SMF type 0/1, note-on/off only, tempo-independent whole-note-fraction Duration units (PR #39), real per-track Voice assignment via interval-graph coloring (PRs #43/#46), flattens multi-track files, fails cleanly on malformed input | T | Yes | Behavioral Contract | RoundTrip
MusicXMLImporter | Adapter | MusicXML → Canonical | music-io | Notation Document Model | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MEIImporter | Adapter | MEI → Canonical | music-io | Notation Document Model | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
ABCImporter | Adapter | ABC → Canonical | music-io | — | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
AudioAnalyzer (importer) | Adapter | Recording → Observations → Canonical | music-io | Recording, Signal Processing | Canonical IR | 17 | Not Started | T | Yes | Behavioral Contract | RoundTrip
OCRImporter | Adapter | Image → Observations → Canonical | music-io | Visual Recognition | Canonical IR | 16 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MIDIRenderer | Adapter | Canonical → MIDI | music-io | Canonical IR | Playback | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MusicXMLRenderer / MusicXmlExport | Adapter | Canonical → MusicXML | music-io | Canonical IR, Notation Doc Model | Export | 14 | **Implemented** (PRs #40–41, `src/export/musicxml_export.*`) — single-part note/measure serialization, `xml:id` on exported notes (for downstream note-highlighting use by Ursatz-GUI); overlap-detection and error-code propagation fixes since applied (PRs #22–25). Row status was "Not Started" as of the 2026-09-09 session — corrected 2026-09-13 | T | Yes | Behavioral Contract | RoundTrip
MEIRenderer | Adapter | Canonical → MEI | music-io | Canonical IR, Notation Doc Model | Export | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
NotationRenderer | Adapter | Canonical → visual notation | music-notation | Score Model | Display/print | 14 | Not Started at kernel level (Ursatz-GUI does this at the application level via OSMD, consuming MusicXmlExport output) | N | Yes | Behavioral Contract | RoundTrip
AudioRenderer | Adapter | Canonical/Performance → Audio | music-performance / music-io | PerformanceEvent | Playback | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
LossModel | Concept | Reports preserved/converted/approximated/discarded/unknown per conversion | music-io | Adapter conversions | Round-trip testing | 14 | Not Started | T | Yes | Structural | RoundTrip

## Corpus

Corpus | Entity | Collection of sources/documents for statistical study | music-corpus | Persistence (EventRepository, InterpretationRepository) | Analyzer, Generator, corpus_stats | 15 | Implemented (minimal — create/list Corpora, add/remove/list piece refs) | B, Q | Yes | Structural | CorpusRegression
Corpus Ingestion / Normalization | Pipeline | Corpus source → Canonical Representation | music-corpus | Importers | Feature Extraction | 15 | Not Started | Q | Yes | Behavioral Contract | CorpusRegression
Corpus Feature Extraction / Indexing | Pipeline | Canonical → features → index | music-corpus | Canonical IR | Statistics | 15 | Not Started | Q | Yes | Behavioral Contract | CorpusRegression
Corpus Statistics / Style Models | Concept | Derived, non-authoritative statistical knowledge | corpus_stats | corpus, persistence, interpretation, identity | Tendency, StatisticalModel | 15 | Implemented (minimal — most-common-chord only) | L, Q | Yes (non-authority rule explicit) | Structural | CorpusRegression
StyleModel | Entity | Frameworks + constraints + preferences + tendencies + prob. models (versioned) | music-theory | Corpus Statistics | Generation | 15 | Not Started | L | Yes | Structural | CorpusRegression
framework-v1-reference-set | Corpus instance | Fixed test corpus defining Framework v1's finite-endpoint criterion | music-corpus | Corpus API | Framework-stage AnalysisRegression (intended, not yet built) | 15 | Registered — 5 pieces: Bach (BWV Anh. 114), Clementi (Op. 36), Field (Nocturne No. 1), Field (Nocturne No. 5), Mozart (K.545). Fixture files in `tests/fixtures/framework-v1-corpus/`. `CorpusRegression` verifies registration ONLY. Manual GUI verification: Bach checked, surfaced two bugs, both since fixed in code (PRs #36, #37) but not re-verified against the corpus. Other 4 pieces never checked. **This remains the actual remaining gap for calling Framework v1 "done"** — see `Overview/Master_Roadmap.md` | Q | Yes | Structural | CorpusRegression

## Plugin / Security

Plugin Contract | Interface | plugin_id, version, capabilities, dependencies, config schema, permissions | music-security / plugin host | Framework/Analyzer/Generator/Importer interfaces | All plugin-provided modules | Post kernel | Not Started | S | Yes | Interface Contract | Security
Plugin Security Model (sandboxing, capability APIs, resource limits) | Concept | Restricts untrusted plugin access | music-security | Plugin Contract | All plugins | Post kernel | Not Started | S | Yes | Behavioral Contract | Security, Fuzz
Declarative Rule Language | Concept | Non-executable rule representation for user-created rules | music-rules | Rule schema | User-authored Frameworks | 9 | Not Started | L, S | Partial ("should preferably" — soft normative) | Structural (recommended) | Security
Query Safety Limits | Concept | Resource limits, pagination, recursion depth, timeout | music-query | Query Interface | Corpus/Relationship traversal | Post kernel | Not Started | Q, S | Yes | Behavioral Contract | Performance, Security

## Validation

Validation Layers (schema, object, domain invariant, relationship, context, framework, constraint, serialization, rendering, security) | Concept | Multi-level validation architecture | cross-cutting | Respective object models | All pipelines | Ongoing | Not Started | R | Yes | Behavioral Contract | Invariant (per-layer)
Domain Invariants (Duration>=0, valid time, valid pitch, unique EntityIDs, required provenance, etc.) | Concept | Enforced object-level rules | music-core | Value/Entity definitions | Validation layer | 1+ | Not Started (as a formal cross-cutting layer — individual invariants are enforced per-object at construction, see `Test_Specification.md`) | R | Yes | Invariant | Invariant
Relationship Invariants | Concept | Endpoint/predicate/context/provenance existence rules | music-relationship | Relationship, PredicateDefinition | Validation layer | 6 | Not Started as a formal layer — Relationship itself is implemented and in real use; these invariants have not been confirmed enforced as a distinct validation pass | R, G | Yes | Invariant | Relationship
Provenance Invariants | Concept | Ancestor/activity/agent existence, DAG acyclicity | music-provenance | Provenance objects | Validation layer | 6 | Not Started | R, K | Yes | Invariant | Provenance

## Testing

Testing Categories (Unit, Property-Based, Invariant, Golden, Serialization, Migration, Round-Trip, Relationship, Provenance, Context, Theory, Transformation, Analysis Regression, Generation Validation, Corpus Regression, Performance, Security, Fuzz) | Concept | Required test strategy coverage | cross-cutting (test infra) | Respective modules under test | All modules | Ongoing | See `Test_Specification.md` for per-category status | U | Yes | Process Contract | N/A (self-defining)
Architecture Tests | Concept | Enforce dependency direction / no theory leakage | cross-cutting (test infra) | Module Architecture | CI gating | Ongoing | Implemented — every analyzer added since 2026-09-09 shipped with its own architecture guard test confirming no upward/sideways layer leakage; full suite 150+ tests as of PR #46 | U | Yes | Process Contract | N/A (self-defining)

## Work / Edition Model

Edition | Entity | Distinct published version of a Work | music-containers | Work | Source Variants | Post kernel | Not Started | B, F | Yes | Structural | Invariant
Arrangement | Entity | Adapted version of a Work | music-containers | Work | — | Post kernel | Not Started | B, F | Yes | Structural | Invariant
Revision | Entity | Version of a Work/Edition | music-containers | Work/Edition, Version | — | Post kernel | Not Started | A, B | Yes | Structural | Invariant
SourceVariant | Value | Variant reading / editorial disagreement | music-containers | Source | Scholarly Annotation | Post kernel | Not Started | B | Partial ("shall eventually support" — deferred normative) | Reference Concept (deferred) | TBD
ScholarlyAnnotation | Entity | Claim/target/author/citation/framework/evidence/uncertainty/status | music-interpretation | Annotation base | Analysis, Provenance | Post kernel | Not Started | I, K | Partial ("should support" — soft) | Reference Concept | TBD

## Observability / Reproducibility (cross-cutting)

Structured Logs / Traces / Metrics | Concept | Observability data, does not alter domain semantics | cross-cutting | — | Ops/monitoring | Ongoing | Not Started | U | Yes | Behavioral Contract | N/A (ops)
Correlation IDs / Analysis IDs / Generation IDs / Provenance IDs | Value | Cross-system tracing identifiers | cross-cutting | EntityID | Observability | Ongoing | Not Started | A, K | Yes | Structural | N/A (ops)
ReproducibilityRecord | Entity | activity_id, input_ids/versions, algorithm/framework version, config, seed, env, output_ids | music-provenance | ProvenanceActivity | Generation, Analysis reproducibility | Ongoing | Not Started | K | Yes | Structural | GenerationValidation, AnalysisRegression
Determinism Contract | Concept | States determinism guarantee or stochastic justification per algorithm | music-analysis / music-generation | Procedure/Generator definitions | Reproducibility | Ongoing | Not Started | K, U | Yes | Behavioral Contract | AnalysisRegression, GenerationValidation

## Performance / Recording

Performance | Entity | Realization of musical material | music-performance | Canonical Events | PerformanceEvent | 2 | Not Started | C, E | Yes | Structural | Invariant
PerformanceEvent | Event | Realized onset/release/velocity/etc., references source NoteEvent(s) | music-performance | NoteEvent (0..n mapping) | Audio rendering, Analysis | 2 | Not Started | C, E | Yes | Structural | Relationship
Recording | Entity | Captured signal (channels, sample rate, encoding, source) | music-io (audio) | DigitalArtifact | Audio pipeline | 17 | Not Started | T | Yes | Structural | Invariant

## Transformation subtypes (base type in `Registry_Foundations.md`)

Transposition | Transformation | Pitch displacement w/ spelling/tuning/range policy | music-transform | NoteEvent(s), Instrument context | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
Inversion | Transformation | Pitch inversion about declared axis | music-transform | NoteEvent(s), axis definition | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
Retrograde | Transformation | Reversal of declared dimension(s) | music-transform | Event sequence | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
RetrogradeInversion | Transformation | Composed retrograde + inversion | music-transform | Retrograde, Inversion | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
Augmentation | Transformation | Temporal scaling (lengthening) | music-transform | Duration/timing dimension flag | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
Diminution | Transformation | Temporal scaling (shortening) | music-transform | Duration/timing dimension flag | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Golden
Fragmentation | Transformation | Extracts sub-material | music-transform | Structure/Event refs | Generated material | 7 | Implemented — confirmed NOT reusable for detection/matching (only clones caller-supplied event ids, no comparison logic); Sentence Detection needed new comparison logic instead | M | Yes | Behavioral Contract | Transformation
Expansion | Transformation | Extends material | music-transform | Structure/Event refs | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation
Contraction | Transformation | Compresses material | music-transform | Structure/Event refs | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation
Displacement | Transformation | Temporal shift | music-transform | TemporalAnchor | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation
Ornamentation | Transformation | One-to-many event derivation | music-transform | NoteEvent, one-to-many provenance mapping | Generated material | 7 | Implemented | M | Yes | Behavioral Contract | Transformation, Provenance
