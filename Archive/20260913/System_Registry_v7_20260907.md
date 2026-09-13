SYSTEM REGISTRY v7 — General-Purpose Computational Music Platform (TDD v5)
Columns: Name | Category | Purpose | Module | Depends On | Depended On By | Phase | Status | Spec Ref | Normative? | Contract Type | Test Spec Ref

Changelog v2 -> v3: Added "CORE REFERENCE TYPES" section formally approving VoiceReference, RelationshipReference, and ProvenanceReference (implemented in commit bf00705 ahead of Registry entry; verified against Registry-level expectations during 2026-09-04 reconciliation pass — no code changes required, no conflicts found). This closes the "PROPOSED ONLY, NOT YET IN REGISTRY" gate referenced in the Domain Specification for these three types.

Changelog v3 -> v3.1: EventRepository and InterpretationRepository
status changed to "Implemented (minimal/personal-scale, feat/
persistence-minimal, merged)". Scope: SQLite-backed storage for
NoteEvent, Uncertainty (3 of 8 kinds), Interpretation (claim persisted
opaquely, never decoded), and thin Chord/Key wrappers keyed by their
embedded Interpretation's EntityID. Does not satisfy the full L9
EntityRepository/RelationshipRepository/StructureRepository/
ProvenanceRepository contracts — those remain Not Started.

Changelog v3.1 -> v4 (2026-09-07 reconciliation): Corrected a confirmed Status-column
lag between this Registry and actual repo state, per the 2026-09-07 self-analysis
reports (Ursatz, ursatz-analyzer, ursatz-gui) and the Reconciliation_Report_20260907.
Rows changed from "Not Started" to "Implemented" (with scope notes) are individually
marked below with "[v4]" in their Status field. This pass covers only rows with direct,
specific evidence (a named function, a working end-to-end pipeline, or explicit spec
confirmation) — it does NOT assert precise status for every row. KNOWN OPEN FOLLOW-UP:
per the Ursatz library's own docs/ARCHITECTURE.md module-status table (verified accurate
by the 2026-09-07 self-analysis), the following modules are marked "Implemented (partial)"
in-repo rather than "Not Started" as most of their rows still read here: pitch, events,
containers, structure, semantics, interpretation, theory, rules, generation, io, and
persistence. Each has a corresponding docs/modules/<name>.md enumerating exactly what's
built vs. deferred. A full per-row reconciliation against those files is recommended as
follow-up work — not performed in this pass, to avoid asserting unverified precision at
the individual-row level for modules with dozens of rows each.

Changelog v4 -> v4.1 (2026-09-07): Resolved Corpus placement (previously
flagged unsettled in Dependency & Layer Spec v3 §78). Corpus is a shared
library module in the Ursatz repo, sibling to Persistence (L9),
cross-cutting — not owned by Analyzer, Generator, or ursatz-gui. Analyzer
and Generator both depend on it directly as peers (same relationship each
already has with Persistence); neither routes through the other. Corpus
depends on Persistence for storage of the pieces it groups. Theory-layer
consumers (Tendency, StatisticalModel) continue to receive only opaque
values from Corpus Statistics — unchanged, already correct.

Changelog v4.1 -> v4.2 (2026-09-09, SUPERSEDED — see v5 below): Framework Output
Contract decision incorrectly relocated Motif, Cadence, Theme, and PhraseOccurrence
from L5 (music-structure) to L6 (music-interpretation). This was made without checking
actual repo state first and was wrong — see v5 changelog entry for the correction. Kept
here only for the audit trail; do not treat v4.2's STRUCTURES/SEMANTIC LAYER shape as
current.

Changelog v4.2 -> v5 (2026-09-09, correction): Reverted v4.2 in full. Motif,
MotifOccurrence, Cadence, Theme, ThemeOccurrence, PhraseOccurrence, and Figure all
confirmed, via direct repo inspection on feat/structures-minimal, to already exist as
plain L5 shells (id + EntityID-reference-list, detection logic deferred to L8) — not as
fixed-vocabulary structs with L6-typed fields, so there was no forward-dependency problem
to resolve in the first place. All seven rows moved back to STRUCTURES / OCCURRENCES
below, with Status corrected to Implemented. The actual working pattern, confirmed across
this session's full Framework-stage branch sequence: a framework-relative claim about an
L5 shell is a generic Interpretation (L6, same type Chord/Key already use) targeting the
shell's EntityID; a claim linking two or more elements uses a Relationship (L6) with the
Interpretation targeting the Relationship's id instead. No new named Interpretation-family
type was created this session for any of these.

Changelog v5 -> v6 (2026-09-09, Framework-stage session close-out): Updated statuses
across every row touched or confirmed this session: Chord (sevenths added), Key, Relationship
+ Predicate Vocabulary (confirmed implemented, real predicates in use: precedes,
transformed_from, resembles), Contour, RhythmicPattern, Voice Leading, Cadence Detection,
Motif Detection, Phrase Segmentation. Added new rows: Cadence Classification, Phrase
Boundary Detection, Period Detection, Sentence Detection, Form Structure Detection,
Thematic Return Detection — none of which previously existed in this Registry at all.
Updated Corpus section with framework-v1-reference-set (5 pieces, registration-only
verification — analysis-output verification against this fixed corpus has NOT yet been
performed; this is the actual remaining open item for calling Framework v1 "done").
Added known-gap notes: chord-region exact-match rigidity confirmed to extend to seventh
chords; TuningSystem/PitchRealization absence confirmed as a concrete (not theoretical)
blocker on Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif
Detection. See Master Roadmap v5 and Dependency & Layer Specification v4 for the full
session narrative and architectural reasoning behind these changes.

PROCESS NOTE (added v6): Registry "Not Started" status lagged actual repo state on the
majority of branches in this session (structures-minimal, semantics-minimal,
voice-leading-analysis, cadence-detection, phrase-segmentation all found real code already
present, or — in structures-minimal's case — a Registry decision that directly
contradicted committed reality). Treat any "Not Started" status in this document as
unverified until a branch's own investigation step confirms it, not as ground truth by
default.

=== IDENTITY / VERSIONING ===
EntityID | Value | Stable identity for persistent objects | music-identity | — | Entity, all persistent objects | 1 | Implemented [v4] | A | Yes | Value Contract | Invariant, Serialization
Version | Value | Distinguish revisions from identity | music-identity | EntityID | Entity, VersionModel | 1 | Implemented [v4] | A | Yes | Value Contract | Invariant
VersionModel | Concept | Defines identity vs revision vs derivation semantics | music-identity | EntityID, Version | Entity, Provenance | 1 | Implemented [v4] | A | Yes | Structural | Invariant
SchemaVersion | Value | Tags schema of serialized/persisted objects | music-identity | — | Serialization, Persistence, Migration | 1 | Implemented [v4] | A, O | Yes | Value Contract | Serialization, Migration

=== CORE REFERENCE TYPES (Registry Sync — Approved v3) ===
VoiceReference | Value | Opaque unowned reference to a Voice (L4), by EntityID | music-core | EntityID | NoteEvent | 1 | Implemented | B | Yes | Value Contract | Invariant
RelationshipReference | Value | Opaque unowned reference to a Relationship (L6), by EntityID | music-core | EntityID | Period Detection, Sentence Detection, Form Structure Detection (real consumers as of v6 — previously "no current consumer as of v3") | 1 | Implemented | B | Yes | Value Contract | Invariant
ProvenanceReference | Value | Opaque unowned reference to Provenance (L6/music-provenance), by EntityID, three-state presence (absent/pending/present) | music-core | EntityID | NoteEvent, Relationship, Interpretation, and other L3-L8 objects requiring provenance | 1 | Implemented | B | Yes | Value Contract | Invariant
FrameworkReference | Value | Opaque unowned reference to a Framework (L7, music-theory), by EntityID — resolves Open Issue §7.1 by letting L6 point at a Framework without depending on the music-theory module itself | music-core | EntityID | Interpretation, Chord, Key, ScaleDegree | 1 | Implemented [v4] — Dependency & Layer Spec v4 confirms this resolution is in effect, not merely proposed | B | Yes | Value Contract | Invariant

NOTE (Open Issue §7.1 resolution, v3): Interpretation-family types no longer depend on Framework directly. They depend on FrameworkReference only (same reference-only pattern as VoiceReference/RelationshipReference/ProvenanceReference). Framework resolution happens at L7+, never below. This closes Dependency & Layer Specification v2 §7.1 — no implementation may hold a live Framework reference at L6.

=== CORE / VALUE OBJECTS ===
Entity | Category (base type) | Identifiable conceptual object | music-core | EntityID, SchemaVersion, Provenance | Work, Instrument, Voice, Motif, Framework, Rule | 1 | Not Started | B | Yes — BOUNDARY CONTESTED (vs Occurrence/Event) | Structural | Invariant
Value | Category (base type) | Immutable semantic content | music-core | — | Duration, MusicalTime, PitchClass, Interval, Tempo | 1 | Not Started | B | Yes | Structural | Property-Based
Occurrence | Category (base type) | Situated instance of a conceptual object | music-core | Structure/Motif/Theme (target) | MotifOccurrence, PhraseOccurrence, ThemeOccurrence | 1 | Not Started | B | Yes — BOUNDARY CONTESTED (vs Entity/Event) | Structural | Invariant, Context
Event | Category (base type) | Occurrence associated with a temporal domain | music-events | Occurrence, TimeAnchor | NoteEvent, RestEvent, ControlEvent | 1 | Not Started | E | Yes — BOUNDARY CONTESTED (vs Occurrence/Entity) | Structural | Invariant
Container | Category (base type) | Groups/organizes objects | music-containers | — | Voice, Layer, Staff, Part, Measure, Phrase, Section, Work | 1 | Not Started | F | Yes | Structural | Invariant
Structure | Category (base type) | Higher-level semantic organization of material | music-structure | Event refs (non-duplicating) | Motif, Figure, Phrase, Theme, Section, Movement, Work | 5 | Not Started | F | Yes | Structural | Invariant
Relationship | Category (base type) | First-class connection between objects | music-relationship | PredicateDefinition, Context, Evidence, Provenance | Query, Analysis, Generation | 6 | Implemented [v6] — confirmed via Period/Sentence/Form Detection, all three real consumers; predicates in real use: precedes, transformed_from, resembles | G | Yes | Structural | Relationship
Assertion | Category (base type) | Claim made by an agent/process | music-interpretation | Agent | Observation, Hypothesis, Interpretation | 4 | Not Started | I | Yes | Structural | Context, Theory
Observation | Category (base type) | Detected/measured info, no interpretation implied | music-interpretation | Source, Method, Uncertainty | Hypothesis, Candidate | 4 | Not Started | I | Yes | Structural | Context
Hypothesis | Category (base type) | Candidate explanation | music-interpretation | Observation, Evidence | Interpretation | 4 | Not Started | I | Yes | Structural | Context
Interpretation | Category (base type) | Contextual claim about musical material | music-interpretation | Context, FrameworkReference, Evidence, Uncertainty, Provenance | Analysis results, Theory evaluation | 4 | Implemented (partial) [v6] — thin-wrapper form confirmed in code (claim persisted opaquely) and used as the sole output mechanism for every analyzer this session (Chord, Key, Voice Leading, Cadence, Motif, Phrase, Period, Sentence, Form); full {target, claim, framework, context, evidence, uncertainty, author, method, status, provenance} shape per TDD not fully confirmed field-by-field | I | Yes | Structural | Context, Theory
Context | Category (base type) | Conditions under which an assertion is evaluated | music-context | Context (parent, composable) | Interpretation, Relationship, Framework evaluation | 4 | Not Started | H | Yes | Structural | Context
Framework | Category (base type) | Interpretive system (theory) | music-theory | Canonical domain interfaces only | Rule, Constraint, Interpretation | 4/9 | Implemented (partial) [v4] — real minimal instance: frameworks/common-practice-minimal, identifies 7 diatonic triads + 4 seventh-chord qualities + major/natural-minor keys | L | Yes | Structural | Theory
Rule | Category (base type) | Formal evaluative statement | music-rules | Framework, Scope | RuleEvaluation | 9 | Not Started | L | Yes | Structural | Theory
Constraint | Category (base type) | Restricts candidate states | music-rules | Framework, Scope | Generation, ConstraintSolver | 9 | Implemented (partial) [v4] — constraint_evaluate() has real null-checking but is a documented placeholder always returning CONSTRAINT_EVAL_NOT_IMPLEMENTED (TDD §96) | L | Yes | Structural | Theory, GenerationValidation
Preference | Category (base type) | Ranks candidates without invalidating | music-rules | Framework | Candidate Evaluation | 9 | Implemented (partial) [v4] — preference_rank() is a similarly documented stub | L | Yes | Structural | Theory
Heuristic | Category (base type) | Strategy/estimate, non-authoritative | music-rules | — | Candidate Evaluation, Generation | 9 | Not Started | L | Yes | Structural | Theory
Tendency | Category (base type) | Observed statistical behavior from corpus | music-theory | Corpus, StatisticalModel | Style Model | 9 | Not Started | L | Yes | Structural | CorpusRegression
Procedure | Category (base type) | Algorithmic method (analysis/generation) | music-analysis / music-generation | Inputs, Config, Version | Analyzer, Generator | 9 | Not Started | L | Yes | Interface Contract | Unit
Activity | Category (base type, PROV) | Process that produces/changes information | music-provenance | Agent | Derivation, Provenance Graph | 6 | Not Started | K | Yes | Structural | Provenance
Evidence | Category (base type) | Structured basis for an assertion | music-interpretation | Observation/source data | Interpretation, RuleEvaluation | 4/6 | Not Started | J | Yes | Structural | Context, Theory
Provenance | Category (base type) | Lineage of derived objects | music-provenance | Entity, Activity, Agent, Derivation | All derived objects | 6 | Not Started | K | Yes | Structural | Provenance
Intent | Category (base type) | Structured compilation of natural-language request | music-generation | Context, Constraints, Preferences | GenerationPlan | 12 | Not Started | M (partial) | Yes | Structural | GenerationValidation
Candidate | Category (base type) | Proposed result pre-acceptance | music-generation / music-analysis | Material, Evaluations | Selection, CandidateEvaluator | 4/12 | Not Started | I, M | Yes | Structural | GenerationValidation
Transformation | Category (base type) | Pure domain operation producing new/derived objects | music-transform | Input type, Parameters, Provenance | Transposition, Inversion, Retrograde, etc. | 7 | Implemented [v4] — all 11 concrete transforms below implemented; confirmed this session (sentence-detection) that Fragmentation is NOT reusable for detection/matching purposes — it only clones caller-supplied event ids, no comparison logic | M | Yes | Behavioral Contract | Transformation, Golden

=== TEMPORAL ARCHITECTURE ===
MusicalTime | Value | Exact symbolic temporal position (rational) | music-time | — | TimeSpan, NoteEvent, MeterMap, TempoMap | 1 | Implemented [v4] — Phase-1 object, see Domain Specification v4 | C | Yes | Value Contract | Property-Based, Golden
Duration | Value | Amount of time (>=0, exact rational) | music-time | — | NoteEvent, RestEvent | 1 | Implemented [v4] — Phase-1 object, see Domain Specification v4 | C | Yes | Value Contract | Property-Based, Invariant
TimeSpan | Value | Explicit start/end + boundary semantics | music-time | MusicalTime | Event, Structure occurrences | 1 | Implemented [v4] — Phase-1 object; boundary vocabulary remains provisional, see Domain Specification v4 | C | Yes | Value Contract | Invariant
TimeInstant | Value | Zero-duration temporal point | music-time | MusicalTime | TempoEvent, MeterChangeEvent | 1 | Not Started | C | Yes | Value Contract | Invariant
NotatedTime / NotatedPosition | Value | Document-relative temporal/graphic position | music-notation | Notation document | Score Model | 2 | Not Started | C, N | Yes | Value Contract | RoundTrip
PerformanceTime | Value | Realized temporal coordinates | music-performance | TempoMap, MusicalTime | PerformanceEvent | 2 | Not Started | C | Yes | Value Contract | Invariant
AudioTime | Value | Position in captured signal space | music-io (audio) | Recording | Audio pipeline | 17 | Not Started | C | Yes | Value Contract | Invariant
TemporalAnchor | Value | Explicit anchor type for objects in time | music-time | MusicalTime/relative refs | Event | 1 | Not Started | C | Yes | Value Contract | Invariant
Meter | Value | Metrical organization (e.g., 4/4) | music-time | — | MeterMap | 2 | Not Started | C | Yes | Value Contract | Unit
MeterMap | Entity | Associates meter with temporal regions | music-time | Meter, TimeSpan | MetricalPosition | 2 | Not Started | C | Yes | Structural | Unit
MetricalPosition | Value | Contextual metrical coordinates | music-time | MeterMap | Analysis, Notation | 2 | Not Started | C | No (descriptive example) | Reference Concept | TBD
Tempo | Value | Tempo modeled independently of symbolic time | music-time | — | TempoMap | 2 | Not Started | C | Yes | Value Contract | Unit
TempoMap | Entity | MusicalTime → PerformanceTime mapping | music-time | MusicalTime, PerformanceTime | Performance rendering | 2 | Not Started | C | Yes | Structural | Unit
TemporalMapping | Concept | First-class mapping between time domains | music-time | MusicalTime, NotatedPosition, PerformanceTime, AudioTime | I/O, Performance | 2 | Not Started | C | Yes | Structural | RoundTrip

=== PITCH ARCHITECTURE ===
PitchIdentity | Value | Abstract pitch info, no tuning assumptions | music-pitch | — | PitchClass, PitchSpelling, PitchRealization | 1 | Implemented [v4] — Phase-1 object, equality resolved v0.3 (strict token identity), see Domain Specification v4. Confirmed this session (voice-leading-analysis, motif-detection-minimal): deliberately opaque, no numeric/tuning semantics — real interval/direction computation from PitchIdentity is not possible without TuningSystem/PitchRealization | D | Yes | Value Contract | Property-Based
PitchClass | Value | Equivalence under declared relation | music-pitch | PitchIdentity, equivalence model | Scale, Interval | 3 | Implemented [v4] — built ahead of Phase 3, holds theory-relative equivalence via caller-supplied relation, see Domain Specification v4 §PitchIdentity | D | Yes | Value Contract | Property-Based
PitchSpelling | Value | Notation-oriented pitch identity (letter/accidental/octave) | music-pitch | PitchIdentity | NoteEvent (optional), Notation | 1 | Implemented [v4] — Phase-1 object, see Domain Specification v4 | D | Yes | Value Contract | Property-Based
PitchRealization | Value | Sounding/physical pitch (frequency, ratio, cents) | music-pitch | TuningSystem | PitchTrajectory, Performance | 3 | Not Started — confirmed this session as a concrete (not theoretical) blocker on Voice Leading, Cadence Classification's PAC/IAC distinction, and Motif Detection all correctly falling back to caller-supplied/Contour-level data instead | D | Yes | Value Contract | Unit
TuningSystem | Entity | Tuning definition (12-TET, JI, custom, etc.) | music-pitch | — | PitchRealization | 3 | Not Started — same blocker note as PitchRealization above | D | Yes | Structural | Unit
PitchTrajectory | Value | Continuous pitch over time (vibrato, glissando) | music-pitch | PitchRealization, time | Performance/Audio analysis | 3 | Not Started | D | Yes | Value Contract | Unit
WrittenPitch | Value | Pitch as notated (pre-transposition) | music-pitch | PitchSpelling | Instrument transposition mapping | 1 | Not Started | D | Yes | Value Contract | Invariant
SoundingPitch | Value | Pitch as sounds (post-transposition) | music-pitch | PitchRealization | Instrument transposition mapping | 1 | Not Started | D | Yes | Value Contract | Invariant
Instrument | Entity | Range, transposition, clefs, technique, etc. | music-events / music-performance | — | InstrumentCapability, Voice, Generation constraints | 1+ | Not Started | E | Partial (property list is illustrative) | Reference Concept | TBD
InstrumentCapability | Value | Range/limits/technique, separable from identity | music-events | Instrument | Generation validation, Practice generation | 1+ | Not Started | E | Yes | Structural | Unit

=== EVENTS ===
NoteEvent | Entity/Event | Symbolic musical occurrence | music-events | EntityID, TemporalAnchor, Duration, PitchIdentity, VoiceReference, ProvenanceReference | Interval, Sonority, PerformanceEvent | 1 | Implemented [v4] — Phase-1 object, see Domain Specification v4; category boundary below remains genuinely unresolved | E | Yes — BOUNDARY UNRESOLVED (Entity vs Event vs pure Occurrence/Value structure) | Structural (Unresolved) | TBD — pending Domain Spec resolution
RestEvent | Event | Explicit symbolic absence | music-events | TemporalAnchor, Duration | Analysis | 1 | Not Started | E | Yes | Structural | Invariant
ControlEvent | Event | Non-note temporal change (pedal, CC, program change) | music-events | TemporalAnchor | Performance rendering | 1 | Not Started | E | Yes | Structural | Unit
Tie | Relationship/Attachment | Connects note events without merging | music-events | NoteEvent (x2) | Notation, Performance realization | 1+ | Not Started | E, G | Yes | Structural | Relationship
Slur | Attachment | Phrasing/legato/analytical grouping | music-events | NoteEvent range | Performance, Notation | 2 | Not Started | E | Yes | Structural | Relationship
Articulation | Attachment | Staccato, tenuto, accent, etc. | music-events | Event | Performance realization | 2 | Not Started | E | Yes | Structural | Unit
Dynamics | Attachment | p, mf, f, cresc./dim. | music-events | Event/TimeSpan | Performance realization | 2 | Not Started | E | Yes | Structural | Unit

=== CONTAINERS ===
Voice | Container | Logical stream/simultaneity domain | music-containers | — | NoteEvent refs, Staff | 1 | Implemented [v4] — Phase-1 object, see Domain Specification v4 | F | Yes | Structural | Invariant
Layer | Container | Notation-layer grouping | music-containers | — | Staff | 2 | Not Started | F | No (mentioned, no dedicated schema) | Reference Concept | TBD
Staff | Container | Notation staff grouping | music-containers | Voice(s), Layer(s) | Part, Score Model | 2 | Not Started | F | Yes | Structural | Invariant
Part | Container | Instrument/performer grouping | music-containers | Staff(s) | Score Model | 2 | Not Started | F | Yes | Structural | Invariant
Measure | Container | Metrical grouping | music-containers | MeterMap | Score Model | 2 | Not Started | F | No (example container) | Reference Concept | TBD
Phrase (container use) | Container | Mid-level grouping | music-containers | Event refs | Structure/Section | 5 | Implemented (partial) [v6] — confirmed as a real, minimal L4 container this session; used only as a transient id-minting source by Phrase Boundary Detection and Form Structure Detection (mirroring how Motif Detection mints MotifOccurrence), not built out to full container semantics | F | No (example container) | Reference Concept | TBD
Section | Container | High-level grouping | music-containers | Phrase refs | Movement/Work | 5 | Implemented (partial) [v6] — confirmed real this session (form-structure-detection); Form Structure Detection uses structural (PAC/IAC-only) cadences to bound sections | F | No (example container) | Reference Concept | TBD
Movement | Container | Work subdivision | music-containers | Section refs | Work | 5 | Not Started | F | No (example container) | Reference Concept | TBD
Work | Entity/Container | Top-level musical work | music-containers | Movement refs | Edition, Arrangement, Performance, Recording | 5 | Not Started | F | Yes | Structural | Invariant

=== STRUCTURES / OCCURRENCES ===
Motif | Structure | Abstract recurring pattern | music-structure | Event refs | MotifOccurrence, SimilarityResult | 5 | Implemented [v6] — confirmed pre-existing this session (feat/structures-minimal); plain L5 shell (id + EntityID-reference-list), detection logic deferred to L8 (Motif Detection). Framework-relative claims about a Motif instance use a generic Interpretation (L6) targeting it — no separate claim-shaped type | F | Yes | Structural | Relationship
MotifOccurrence | Occurrence | Situated instance of a motif | music-structure | Motif, region/events | Motif Analysis results | 5 | Implemented [v6] — confirmed pre-existing; references Motif via a bare EntityID (not RelationshipReference — confirmed correct as committed, not reworked) | F | Yes | Structural | Relationship
Figure | Structure | Small structural unit | music-structure | Event refs | Phrase | 5 | Implemented (partial) [v6] — confirmed pre-existing this session, same L5 shell shape as Motif/Theme/Cadence | F | No (example only) | Reference Concept | TBD
Theme | Structure | Structural unit above phrase | music-structure | Phrase refs | Section | 5 | Implemented [v6] — confirmed pre-existing this session; L5 shell, consumed by Form Structure/Thematic Return Detection via ThemeOccurrence | F | No (example only) | Reference Concept | TBD
ThemeOccurrence | Occurrence | Situated instance of a theme | music-structure | Theme, region | Formal analysis | 5 | Implemented [v6] — confirmed pre-existing; Thematic Return Detection links ThemeOccurrence ids directly (not Section ids) via a "resembles" Relationship — the recurrence claim is about the material, not its container | F | No (example only) | Reference Concept | TBD
Cadence | Structure | Formal closure unit | music-structure | Event refs, Harmonic context | Formal Analysis, Phrase Segmentation | 5/8 | Implemented [v6] — confirmed pre-existing this session; L5 shell, targeted by Cadence Detection/Classification's Interpretation output | F | Yes (explicit Phase 5/8 deliverable) | Structural | AnalysisRegression
PhraseOccurrence | Occurrence | Situated instance of a phrase | music-structure | Phrase, region | Formal Analysis | 5 | Implemented [v6] — confirmed pre-existing this session; constructed by Phrase Boundary Detection via a transient Phrase (L4) id-minting pattern, same shape MotifDetection uses for MotifOccurrence | F | No (naming-pattern example) | Reference Concept | TBD

NOTE (v6, corrects v4.2): an earlier Framework Output Contract decision incorrectly relocated
Motif, Cadence, Theme, and PhraseOccurrence to L6 as new Interpretation-shaped types. That
was wrong — all seven rows above were already committed as plain L5 shells before the
decision was made, and the decision was reached without checking repo state first. See
changelog v4.2 -> v5 above for the correction, and Dependency & Layer Specification v4 for
the full architectural reasoning. The durable pattern going forward: L5 shell (the thing) +
a generic L6 Interpretation targeting it (a framework-relative claim about it) — same
relationship Chord already has to Sonority. Where a claim spans two or more elements
(Period, Sentence, Form/Thematic Return), a Relationship (L6) links them and the
Interpretation targets the Relationship's id instead.

=== SEMANTIC LAYER ===
Interval | Value | Directed/undirected/diatonic/chromatic distance | music-semantics | PitchIdentity/PitchClass pair | Sonority, Contour, Analysis | 3 | Not Started | D | Yes | Value Contract | Property-Based
Sonority | Value | Selected collection of simultaneous pitch-bearing events | music-semantics | Events, selection semantics | Chord Interpretation | 3 | Not Started | D | Yes | Value Contract | Golden
Chord | Interpretation | Framework-specific interpretation of a Sonority | music-interpretation | Sonority, FrameworkReference, Context | Harmonic Analysis | 8 | Implemented (minimal) [v6] — thin wrapper (Persistence, feat/persistence-minimal) + real production via Chord Identification, now including 4 seventh-chord qualities (dominant7, major7, minor7, half-diminished7 — fully-diminished7 deliberately excluded, requires harmonic minor, out of scope) alongside the original 7 diatonic triads, quality-keyed (root-relative interval set) not degree-keyed | I, L | Yes | Behavioral Contract | Theory
Key | Interpretation | Contextual claim, not intrinsic to notes | music-interpretation | Context, FrameworkReference | Scale Degree, Roman Numeral Analysis | 8 | Implemented (minimal) [v6] — thin wrapper (Persistence) + real production via Key Estimation, major/natural-minor only | I, L | Yes | Behavioral Contract | Theory
Scale | Value | Ordered pitch collection under declared semantics | music-semantics | PitchClass, TuningSystem, Framework | Scale Degree, Scale Analysis | 3 | Not Started | D | No (attributes listed as "possible") | Reference Concept | TBD
ScaleDegree | Interpretation | Contextual mapping (pitch → degree) | music-interpretation | Scale, Context, FrameworkReference | Roman Numeral Analysis | 3/8 | Not Started | D, I | Yes | Structural | Theory
Contour | Value | Relative pitch change (up/down/same) | music-semantics | PitchIdentity sequence | Motif/Similarity Analysis | 3 | Implemented [v6] — confirmed this session (feat/semantics-minimal); direction is caller-supplied, not computed from PitchIdentity (deliberate — PitchIdentity is opaque, same constraint as Voice Leading). This makes Contour-based matching transposition-invariant by construction, which Motif Detection relies on for v1's exact-repetition/transposition-only scope | D | No (descriptive examples) | Reference Concept | TBD
RhythmicPattern | Value | Independently representable rhythm sequence | music-semantics | Duration/TimeSpan sequence | Pattern Matching, Motif Detection | 3 | Implemented [v6] — confirmed this session (feat/semantics-minimal); fully constructible/queryable from rhythm alone (no pitch header dependency), per TDD requirement | C, D | Yes | Structural | Unit

=== RELATIONSHIP MODEL ===
PredicateDefinition | Entity | Metadata for a relationship predicate | music-relationship | — | Relationship | 6 | Implemented [v6] — confirmed via real Relationship usage this session | G | Yes | Structural | Relationship
Predicate Vocabulary (contains, precedes, derived_from, realizes, resembles, interpreted_as, etc.) | Value set | Initial extensible predicate list | music-relationship | PredicateDefinition | Relationship | 6 | Implemented (partial) [v6] — 3 of the documented vocabulary confirmed in real use this session: "precedes" (sequential order — Period antecedent→consequent, Form section order), "transformed_from" (derivation — Sentence statement→repetition→continuation, direction: derived-part→source), "resembles" (recurrence, no derivation direction claimed — Form thematic return). Extend this vocabulary before inventing new predicate names for a genuinely new relationship kind | G | No (explicitly extensible, not fixed) | Reference Concept | TBD

=== PROVENANCE ===
ProvenanceActivity | Entity (PROV) | Process producing/changing info (import, analysis, transform, generation) | music-provenance | Agent | Derivation | 6 | Not Started | K | Yes | Structural | Provenance
ProvenanceAgent | Entity (PROV) | Human/software/org/ML model/external source | music-provenance | — | Activity, Derivation | 6 | Not Started | K | Yes | Structural | Provenance
Derivation | Entity (PROV) | Output/input/activity/params/agent/version record | music-provenance | Activity, Agent | Provenance Graph | 6 | Not Started | K | Yes | Structural | Provenance
ProvenanceGraph | Concept | Acyclic lineage graph | music-provenance | Derivation(s) | Audit/Reproducibility | 6 | Not Started | K | Yes | Invariant | Provenance
AuditEvent | Entity | Who changed what/when/why (distinct from provenance) | music-provenance | Agent, timestamp | Audit History | 6 | Not Started | K | Yes | Structural | Provenance
Annotation | Entity | Human-authored analysis/claim | music-interpretation | Author, Target, Context | Promotion to authored content | 4/6 | Not Started | I, K | Yes | Structural | Context

=== EQUALITY / CANONICALIZATION ===
IdentityEquality | Concept | Same entity/version | music-core | EntityID, Version | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
ValueEquality | Concept | Semantic value equality | music-core | Value objects | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
StructuralEquality | Concept | Equivalent structure under rules | music-core | Structure rules | Equality checks | 1 | Not Started | A | Yes | Invariant | Invariant
SemanticEquality | Concept | Equivalent despite differing representation | music-core | Context/Tuning | Canonicalization | 1 | Not Started | A | Yes | Invariant | Invariant
CanonicalEquality | Concept | Same canonical normal form | music-core | Normalization Policy (versioned) | Serialization, Query | 1 | Not Started | A, O | Yes | Invariant | Serialization

=== CANONICAL IR / SERIALIZATION / PERSISTENCE ===
Canonical Musical Model / IR | Concept | Theory-neutral representation of musical content | music-core | Event, Structure, Relationship, Context, Provenance | Everything downstream | 1 | Not Started | N | Yes | Structural | Serialization, Invariant
Serialization Format(s) (JSON, protobuf, CBOR, XML, binary) | Adapter | Canonical object → external syntax | music-io | Canonical IR | Persistence, I/O | 1+ | Not Started | O | Yes (prohibition normative; specific formats are examples) | Behavioral Contract | Serialization
Canonical Serialization Spec | Concept | Schema version, stable IDs, deterministic ordering, exact rationals | music-io | Serialization | Migration | 1 | Not Started | O | Yes | Structural | Serialization
EventRepository | Interface | Repository for Event objects | music-persistence | Canonical IR | Application layer | 1 | Implemented (minimal/personal-scale, feat/persistence-minimal, merged) | P | Yes | Interface Contract | Unit
EntityRepository | Interface | Repository for Entity objects | music-persistence | Canonical IR | Application layer | 1 | Not Started | P | Yes | Interface Contract | Unit
RelationshipRepository | Interface | Repository for Relationship objects | music-persistence | Relationship | Application layer | 6 | Not Started in Ursatz proper. ursatz-gui (PR #13, 2026-09-09) needed real persistence for Relationship instances produced by Period/Sentence/Form/Thematic Return Detection and could not wait on this, so it added an ursatz-gui-owned SQLite table mirroring the existing kIndexTableName/kMetaTableName pattern — application-layer stopgap, not a substitute for a real RelationshipRepository. Retire that table in favor of this interface once/if it's built | P | Yes | Interface Contract | Unit
StructureRepository | Interface | Repository for Structure objects | music-persistence | Structure | Application layer | 5 | Not Started — same persistence-confirmation caveat as RelationshipRepository above, for Motif/Cadence/Theme/PhraseOccurrence L5 shells | P | Yes | Interface Contract | Unit
InterpretationRepository | Interface | Repository for Interpretation objects | music-persistence | Interpretation | Application layer | 4 | Implemented (minimal/personal-scale, feat/persistence-minimal, merged) | P | Yes | Interface Contract | Unit
ProvenanceRepository | Interface | Repository for Provenance objects | music-persistence | Provenance | Application layer | 6 | Not Started | P | Yes | Interface Contract | Unit
Cache | Concept | Derived-value cache keyed by semantic dependencies | music-persistence | object_id, version, analyzer/framework version, context/config hash | Query performance | Post-Phase 1 | Not Started | P | Yes (key composition explicit) | Behavioral Contract | Performance

=== QUERY ===
Query Interface (find_notes, find_intervals, find_occurrences, find_interpretations, find_relationships) | Interface | Domain queries replacing raw SQL | music-query | Repositories | Applications | Post kernel | Not Started | Q | Yes | Interface Contract | Unit
PatternMatching | Concept | Match over declared dimensions (interval, rhythm) | music-query | Interval/RhythmicPattern | Motif Detection, Search | 8 | Not Started — Motif Detection (implemented, v6) deliberately does NOT depend on this; it does direct Contour/RhythmicPattern sequence comparison instead, per the Framework-stage session's scope decision to avoid standing up unbuilt L9 query infrastructure | Q | Yes | Behavioral Contract | Unit
SimilarityModel | Concept | Explicit-dimension similarity (pitch, contour, rhythm, transposition, etc.) | music-query | Interval, Contour, RhythmicPattern | SearchResult | 8 | Not Started — same scope-avoidance note as PatternMatching above | Q | Yes (no undeclared-similarity prohibition is explicit) | Behavioral Contract | Unit
SearchResult | Value | Target, match region, similarity, criteria, evidence, provenance | music-query | PatternMatching/SimilarityModel | Application UI | 8 | Not Started | Q | Yes | Structural | Unit
Views (Temporal, Pitch, Rhythmic, Voice, Harmonic, Motivic, Formal, Performance, Notation) | Concept | Derived, non-duplicating projections of canonical data | music-query | Canonical IR | Applications | Post kernel | Not Started | Q | Yes | Structural | Invariant

=== TRANSFORMATION ===
Transposition | Transformation | Pitch displacement w/ spelling/tuning/range policy | music-transform | NoteEvent(s), Instrument context | Generated material | 7 | Implemented [v4] — L8-Transform fully implemented, tested, valgrind-clean per repo self-analysis | M | Yes | Behavioral Contract | Transformation, Golden
Inversion | Transformation | Pitch inversion about declared axis | music-transform | NoteEvent(s), axis definition | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Golden
Retrograde | Transformation | Reversal of declared dimension(s) | music-transform | Event sequence | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Golden
RetrogradeInversion | Transformation | Composed retrograde + inversion | music-transform | Retrograde, Inversion | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Golden
Augmentation | Transformation | Temporal scaling (lengthening) | music-transform | Duration/timing dimension flag | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Golden
Diminution | Transformation | Temporal scaling (shortening) | music-transform | Duration/timing dimension flag | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Golden
Fragmentation | Transformation | Extracts sub-material | music-transform | Structure/Event refs | Generated material | 7 | Implemented [v4] — see Transposition note. Confirmed this session (feat/sentence-detection): NOT reusable for detection/matching — it only clones caller-supplied event ids, no comparison logic. Sentence Detection needed new comparison logic (SentenceContinuationMatching) instead | M | Yes | Behavioral Contract | Transformation
Expansion | Transformation | Extends material | music-transform | Structure/Event refs | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation
Contraction | Transformation | Compresses material | music-transform | Structure/Event refs | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation
Displacement | Transformation | Temporal shift | music-transform | TemporalAnchor | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation
Ornamentation | Transformation | One-to-many event derivation | music-transform | NoteEvent, one-to-many provenance mapping | Generated material | 7 | Implemented [v4] — see Transposition note | M | Yes | Behavioral Contract | Transformation, Provenance

=== ANALYSIS ===
Analyzer (interface) | Interface | analyze(input, context, config) → AnalysisResult | music-analysis | Canonical IR, Context, Framework | Analysis pipeline | 8 | Not Started | I, L | Yes | Interface Contract | AnalysisRegression
FeatureExtraction | Concept | Explicit derived features (interval, contour, density, etc.) | music-analysis | Canonical IR | Candidate Generation | 8 | Not Started | I | Yes | Structural | AnalysisRegression
Interval Analysis | Procedure | Analyzer | music-analysis | Interval | Interpretation | 8 | Not Started | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Scale Analysis | Procedure | Analyzer | music-analysis | Scale | Interpretation | 8 | Not Started | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Chord Identification | Procedure | Analyzer | music-analysis | Sonority, Framework | Interpretation | 8 | Implemented (minimal) [v6] — wired to common-practice-minimal framework, verified against real commercial MIDI (Pachelbel's Canon) for the original 7 triads; now also identifies 4 seventh-chord qualities (v6, feat/chord-vocab-sevenths), verified at unit level only so far. Compiles into frameworks/common-practice-minimal, not libursatz itself. KNOWN GAP: chord-region exact-match rigidity (no tolerance for extra non-chord tones) confirmed this session to extend to seventh-chord matching too, inherited by construction — not fixed | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Key Estimation | Procedure | Analyzer | music-analysis | Sonority, Context, Framework | Interpretation | 8 | Implemented (minimal) [v4] — same framework/verification as Chord Identification; major/natural-minor only, containment-matching fix applied for real-world data | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Voice Leading (analysis) | Procedure | Analyzer | music-analysis | NoteEvent sequences, Framework | Interpretation | 8/11 | Implemented (packaging-only) [v6] — feat/voice-leading-analysis pre-existed this session's branch as caller-supplied-classification packaging (motion/interval-perfection); this session added a bass_motion field (step/leap/same) plus regression tests. Does NOT compute real values from pitch data itself — blocked on TuningSystem/PitchRealization (deliberate, PitchIdentity is opaque by design). Real computation, if ever added, belongs in a consumer app (ursatz-analyzer's pitch_class_util or a sibling), not the kernel | L | Yes (Phase 8/11 deliverable) | Behavioral Contract | AnalysisRegression
Cadence Detection | Procedure | Analyzer | music-analysis | Harmonic context | Interpretation (targeting the L5 Cadence shell) | 8 | Implemented [v6] — feat/cadence-detection pre-existed as a generic caller-supplied-candidate packager; this session added an optional Framework param threading real FrameworkReference into its output | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Cadence Classification | Procedure | Analyzer | music-analysis | Chord Identification (root-motion), optional Voice Leading bass-motion, optional soprano_resolves_to_tonic | Interpretation (via Cadence Detection delegation) | 8 | Implemented [v6, new row] — derives PAC/IAC/HC/Plagal/Deceptive from real chord root-motion (scale-degree roots, not parsed from chord label strings). PAC vs. IAC requires soprano-resolution data not computed anywhere in the codebase; reports "authentic (unclassified)" rather than guessing. from_is_dominant_seventh and Voice Leading bass-motion are optional, confidence-weight-only refinements, never required | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Motif Detection | Procedure | Analyzer | music-analysis | Contour, RhythmicPattern (direct comparison, not PatternMatching/SimilarityModel) | MotifOccurrence, Interpretation | 8 | Implemented [v6] — feat/motif-detection-minimal pre-existed as caller-supplied-occurrence packaging; this session added the actual Contour+RhythmicPattern sequence-matching step (exact equality over caller-supplied windows). Covers exact repetition and transposition only (transposition-invariant by construction, since Contour is direction-only) — inversion/retrograde/fragmentation matching deferred to v2, pull forward only if the fixed test corpus requires it. Requires >=2 occurrences to count as a match; every match reported at fixed 1/1 confidence (no partial-credit dimension exists in v1) | L, Q | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Phrase Segmentation | Procedure | Analyzer | music-analysis | Caller-supplied is_boundary per event | Interpretation (targeting NoteEvent directly) | 8 | Implemented [v6] — pre-existed this session's branches entirely (predates Cadence/Motif/Voice Leading Detection), as a caller-supplied-classification packager. Left deliberately untouched this session — see Phrase Boundary Detection below for the real detection logic, added as a new analyzer on top rather than a rework | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Phrase Boundary Detection | Procedure | Analyzer | music-analysis | Cadence Detection/Classification (primary signal), Contour/RhythmicPattern discontinuity (secondary, default max_length_without_cadence = 8) | Two Interpretations per boundary point (delegated plain-boundary claim via Phrase Segmentation, unchanged; plus a second Interpretation recording which signal fired + cadence type) + PhraseOccurrence (via transient Phrase L4 id-minting) | 8 | Implemented [v6, new row] — feat/phrase-segmentation. Both Interpretations target the same point deliberately — same "competing/complementary interpretations coexist" principle Chord/Key already use, not a workaround | L | Yes (Phase 8 deliverable) | Behavioral Contract | AnalysisRegression
Period Detection | Procedure | Analyzer | music-analysis | PhraseOccurrence sequence, Phrase Boundary Detection's cadence-type signal | Interpretation (targeting a Relationship, predicate "precedes", source=antecedent/target=consequent) | 5/8 | Implemented [v6, new row] — feat/period-detection. First real use of Relationship from the analysis layer. Cadence-strength ordering (HC < deceptive < plagal < authentic-unclassified < authentic-IAC < authentic-PAC) is a stated default, not Registry-settled. Pairs touching an absent or "unclassified" cadence are skipped, not guessed at | L | Yes (explicit Phase 5/8 deliverable) | Behavioral Contract | AnalysisRegression
Sentence Detection | Procedure | Analyzer | music-analysis | Motif Detection's exact-repetition/transposition matches, continuation material (Contour/RhythmicPattern prefix match, min 2 events, strictly shorter than statement) | Interpretation, via two chained Relationships (predicate "transformed_from", direction derived-part→source): statement→repetition, repetition→continuation | 5/8 | Implemented [v6, new row] — feat/sentence-detection. Confirmed Fragmentation transform not reusable for detection (see Transformation section) — new SentenceContinuationMatching logic built instead. A use-after-free was found (via the branch's own regression test) and fixed within the branch before merge; full suite re-run clean afterward | L | Yes (explicit Phase 5/8 deliverable) | Behavioral Contract | AnalysisRegression
Form Structure Detection | Procedure | Analyzer | music-analysis | PhraseOccurrence/Period/Sentence sequence, structural (PAC/IAC only — the only cadence types that reach tonic) cadence boundaries | Interpretation (targeting a Relationship, predicate "precedes", section order) | 5/8 | Implemented [v6, new row] — feat/form-structure-detection. Only "authentic (PAC)" and "authentic (IAC)" count as section-closing; all other cadence types (including authentic-unclassified) are phrase-level only, per the same unknown-vs-low-confidence discipline used throughout this session | L | Yes (explicit Phase 5/8 deliverable) | Behavioral Contract | AnalysisRegression
Thematic Return Detection | Procedure | Analyzer | music-analysis | Theme/ThemeOccurrence, Motif Detection's Contour+RhythmicPattern comparison reused at Theme granularity | Interpretation (targeting a Relationship, predicate "resembles") | 5/8 | Implemented [v6, new row] — feat/form-structure-detection (same branch as Form Structure Detection). Links ThemeOccurrence ids directly, not Section ids — the recurrence claim is about the material, not its container | L | Yes (explicit Phase 5/8 deliverable) | Behavioral Contract | AnalysisRegression

=== GENERATION ===
IntentCompiler | Component | Natural language → structured Intent | music-generation | NLP layer, Intent schema | Constraint Extraction | 12 | Not Started | M (Intent) | Yes | Behavioral Contract | GenerationValidation
ConstraintExtraction/Normalization | Concept | Intent → executable constraint set | music-generation | Intent | Conflict Detection | 12 | Not Started | L, M | Yes | Behavioral Contract | GenerationValidation
ConflictDetection | Concept | Reports constraint conflicts explicitly | music-generation | Constraint set | GenerationPlan | 12 | Not Started | L | Yes | Behavioral Contract | GenerationValidation
GenerationPlan | Entity | Structured plan feeding generators | music-generation | Intent, Constraints, Preferences | Generator | 12 | Not Started | M | Yes | Structural | GenerationValidation
Generator (interface) | Interface | generate(plan) → CandidateSet | music-generation | GenerationPlan | ThemeGenerator, PhraseGenerator, etc. | 12 | Not Started | M | Yes | Interface Contract | GenerationValidation
CandidateEvaluator | Component | Evaluates candidates vs rules/constraints/prefs/models | music-generation | Rule, Constraint, Preference, StatisticalModel | Selection | 12 | Not Started | L | Yes | Behavioral Contract | GenerationValidation
ConstraintSolver | Component | Solves/searches constraint space | music-generation | Constraint set | Candidate Generation | 12 | Not Started | M | No (Phase-list mention only, no dedicated contract) | Reference Concept | TBD
Search / Optimizer | Component | Search strategy over candidates | music-generation | CandidateSet | Selection | 12 | Not Started | M | No (Phase-list mention only) | Reference Concept | TBD
GenerationRecord | Entity | Records seed, versions, config for reproducibility | music-generation | Provenance | Reproducibility Record | 12 | Not Started | K, M | Yes | Structural | GenerationValidation
Hierarchical Generators (ThemeGenerator, PhraseGenerator, BasicIdeaGenerator) | Component | Composable generation levels | music-generation | Generator interface | Section/Work-level generation | 13 | Not Started | M | Yes | Behavioral Contract | GenerationValidation
StatisticalModel | Entity | ML/statistical output object (features, prob, training corpus, version) | music-theory / ML boundary | Corpus, training pipeline | Tendency, CandidateEvaluator | 9 | Not Started | L | Yes | Structural | CorpusRegression
PracticeRequest / PracticeGenerator | Component | Instrument/technique/skill-driven generation | music-generation | InstrumentCapability, Difficulty Model | Canonical Musical Representation | Practice app | Not Started | M | Yes | Structural | GenerationValidation
DifficultyModel | Concept | Multidimensional difficulty (pitch, rhythm, range, technique, etc.) | music-generation | InstrumentCapability | Practice Generation | Practice app | Not Started | M | Yes (multidimensionality explicit; dimension list illustrative) | Structural | Unit
IntervalRequest (Ear Training) | Component | Structured ear-training request | music-generation | Interval | Ear Training app | App layer | Not Started | M | No (example schema) | Reference Concept | TBD

=== PERFORMANCE / RECORDING ===
Performance | Entity | Realization of musical material | music-performance | Canonical Events | PerformanceEvent | 2 | Not Started | C, E | Yes | Structural | Invariant
PerformanceEvent | Event | Realized onset/release/velocity/etc., references source NoteEvent(s) | music-performance | NoteEvent (0..n mapping) | Audio rendering, Analysis | 2 | Not Started | C, E | Yes | Structural | Relationship
Recording | Entity | Captured signal (channels, sample rate, encoding, source) | music-io (audio) | DigitalArtifact | Audio pipeline | 17 | Not Started | T | Yes | Structural | Invariant

=== NOTATION / SCORE ===
Score Model (Staff, Clef, KeySignature, TimeSignature, Measure, Beam, Stem, Barline, Tuplet, Layout, Page, System) | Concept group | Specialized representation layer, non-authoritative over canonical events | music-notation | Canonical Events, Containers | Notation rendering | 2 | Not Started | F, N | Yes (non-authority rule explicit; internal item list illustrative) | Structural | Invariant
Notation Document Model | Concept | Preserves engraving/layout/editorial info without canonical-event equivalent | music-notation | DigitalArtifact | MusicXML/MEI adapters | 2/14 | Not Started | N | Yes | Structural | RoundTrip

=== WORLD / SOURCE / DIGITAL ARTIFACT ===
Source | Entity | External artifact/phenomenon with identity/provenance | (cross-cutting) | — | DigitalArtifact, Recording | 1 | Not Started | B | Yes | Structural | Provenance
DigitalArtifact | Entity | External digital object (PDF, MusicXML, MIDI, MEI, WAV, JSON, image, DB export) | music-io | Source | Import pipeline | 1 | Not Started | T | Yes | Structural | Invariant

=== I/O ADAPTERS ===
MIDIImporter | Adapter | MIDI → Observations/Candidates → Canonical | music-io | MIDI transport concepts | Canonical IR | 14 | Implemented (minimal) [v4] — SMF type 0/1, note-on/off only, fixed 120 BPM assumption, flattens multi-track files, fails cleanly on malformed input | T | Yes | Behavioral Contract | RoundTrip
MusicXMLImporter | Adapter | MusicXML → Canonical | music-io | Notation Document Model | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MEIImporter | Adapter | MEI → Canonical | music-io | Notation Document Model | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
ABCImporter | Adapter | ABC → Canonical | music-io | — | Canonical IR | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
AudioAnalyzer (importer) | Adapter | Recording → Observations → Canonical | music-io | Recording, Signal Processing | Canonical IR | 17 | Not Started | T | Yes | Behavioral Contract | RoundTrip
OCRImporter | Adapter | Image → Observations → Canonical | music-io | Visual Recognition | Canonical IR | 16 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MIDIRenderer | Adapter | Canonical → MIDI | music-io | Canonical IR | Playback | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MusicXMLRenderer | Adapter | Canonical → MusicXML | music-io | Canonical IR, Notation Doc Model | Export | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
MEIRenderer | Adapter | Canonical → MEI | music-io | Canonical IR, Notation Doc Model | Export | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
NotationRenderer | Adapter | Canonical → visual notation | music-notation | Score Model | Display/print | 14 | Not Started | N | Yes | Behavioral Contract | RoundTrip
AudioRenderer | Adapter | Canonical/Performance → Audio | music-performance / music-io | PerformanceEvent | Playback | 14 | Not Started | T | Yes | Behavioral Contract | RoundTrip
LossModel | Concept | Reports preserved/converted/approximated/discarded/unknown per conversion | music-io | Adapter conversions | Round-trip testing | 14 | Not Started | T | Yes | Structural | RoundTrip

=== CORPUS ===
Corpus | Entity | Collection of sources/documents for statistical study | music-corpus | Persistence (EventRepository, InterpretationRepository) | Analyzer, Generator, corpus_stats | 15 | Implemented (minimal — create/list Corpora, add/remove/list piece refs, feat/corpus-minimal + feat/corpus-api, merged) | B, Q | Yes | Structural | CorpusRegression
Corpus Ingestion / Normalization | Pipeline | Corpus source → Canonical Representation | music-corpus | Importers | Feature Extraction | 15 | Not Started | Q | Yes | Behavioral Contract | CorpusRegression
Corpus Feature Extraction / Indexing | Pipeline | Canonical → features → index | music-corpus | Canonical IR | Statistics | 15 | Not Started | Q | Yes | Behavioral Contract | CorpusRegression
Corpus Statistics / Style Models | Concept | Derived, non-authoritative statistical knowledge | corpus_stats | corpus, persistence, interpretation, identity | Tendency, StatisticalModel | 15 | Implemented (minimal — most-common-chord only, feat/corpus-stats-minimal, merged) | L, Q | Yes (non-authority rule explicit) | Structural | CorpusRegression
StyleModel | Entity | Frameworks + constraints + preferences + tendencies + prob. models (versioned) | music-theory | Corpus Statistics | Generation | 15 | Not Started | L | Yes | Structural | CorpusRegression

framework-v1-reference-set | Corpus instance | Fixed test corpus defining Framework v1's finite-endpoint criterion | music-corpus | Corpus API (feat/corpus-minimal) | Framework-stage AnalysisRegression (intended, not yet built) | 15 | Registered [v6] — 5 pieces: Bach (BWV Anh. 114), Clementi (Op. 36), Field (Nocturne No. 1), Field (Nocturne No. 5), Mozart (K.545). Fixture files live in tests/fixtures/framework-v1-corpus/. CorpusRegression test verifies registration ONLY — does not run analysis and check output. Manual GUI verification pass started 2026-09-09 (all 7 analyzer types now wired into upload pipeline as of PR #13, previously blocked): Bach checked, surfaced two open analysis-quality issues not yet triaged — (1) only one chord Interpretation persists for the entire 430-note piece, (2) thematic-return matching produces dense near-total pairwise matches among phrase-boundary occurrences, suggesting a threshold/window bug. Clementi, Field No.1, Field No.5, Mozart not yet checked (or re-checked, Mozart). THIS REMAINS THE ACTUAL REMAINING GAP for calling Framework v1 "done" | Q | Yes | Structural | CorpusRegression

=== THEORY FRAMEWORKS (independent modules) ===
First–Fifth Species Counterpoint | Framework | Independent framework module | music-theory (plugin) | Framework base, Rule, Constraint | Generation validation (counterpoint) | 10 | Not Started | L | Yes (explicit Phase 10 deliverable, kernel-isolation rule) | Behavioral Contract (Plugin) | Theory
Common-Practice Harmony (Roman Numeral, Harmonic Function, Voice Leading, Cadence, Tonicization, Modulation) | Framework | Independent framework module | music-theory (plugin) | Framework base, Context, Interpretation | Harmonic Analysis | 11 | Not Started — confirmed this session (feat/voice-leading-analysis) that Voice Leading and Cadence Detection/Classification were built directly against frameworks/common-practice-minimal (Phase 8), NOT this separate Phase 11 plugin framework module. This full plugin module remains unbuilt; do not assume it exists just because Voice Leading/Cadence work does | L | Yes (explicit Phase 11 deliverable) | Behavioral Contract (Plugin) | Theory
Schoenberg / Persichetti / Jazz Harmony / Neo-Riemannian / Schenkerian / Set Theory / Modal Theory / Microtonality / Contemporary Systems | Framework | Advanced, independently pluggable frameworks | music-theory (plugin) | Framework base | Analysis/Generation (opt-in) | 18 | Not Started | L | No (named as examples; explicitly optional/no kernel dependency) | Reference Concept (optional plugin) | TBD

=== PLUGIN / SECURITY ===
Plugin Contract | Interface | plugin_id, version, capabilities, dependencies, config schema, permissions | music-security / plugin host | Framework/Analyzer/Generator/Importer interfaces | All plugin-provided modules | Post kernel | Not Started | S | Yes | Interface Contract | Security
Plugin Security Model (sandboxing, capability APIs, resource limits) | Concept | Restricts untrusted plugin access | music-security | Plugin Contract | All plugins | Post kernel | Not Started | S | Yes | Behavioral Contract | Security, Fuzz
Declarative Rule Language | Concept | Non-executable rule representation for user-created rules | music-rules | Rule schema | User-authored Frameworks | 9 | Not Started | L, S | Partial ("should preferably" — soft normative) | Structural (recommended) | Security
Query Safety Limits | Concept | Resource limits, pagination, recursion depth, timeout | music-query | Query Interface | Corpus/Relationship traversal | Post kernel | Not Started | Q, S | Yes | Behavioral Contract | Performance, Security

=== VALIDATION ===
Validation Layers (schema, object, domain invariant, relationship, context, framework, constraint, serialization, rendering, security) | Concept | Multi-level validation architecture | cross-cutting | Respective object models | All pipelines | Ongoing | Not Started | R | Yes | Behavioral Contract | Invariant (per-layer)
Domain Invariants (Duration>=0, valid time, valid pitch, unique EntityIDs, required provenance, etc.) | Concept | Enforced object-level rules | music-core | Value/Entity definitions | Validation layer | 1+ | Not Started | R | Yes | Invariant | Invariant
Relationship Invariants | Concept | Endpoint/predicate/context/provenance existence rules | music-relationship | Relationship, PredicateDefinition | Validation layer | 6 | Not Started — flagged given Relationship itself is now confirmed Implemented; these invariants have not been confirmed enforced anywhere | R, G | Yes | Invariant | Relationship
Provenance Invariants | Concept | Ancestor/activity/agent existence, DAG acyclicity | music-provenance | Provenance objects | Validation layer | 6 | Not Started | R, K | Yes | Invariant | Provenance

=== TESTING ===
Testing Categories (Unit, Property-Based, Invariant, Golden, Serialization, Migration, Round-Trip, Relationship, Provenance, Context, Theory, Transformation, Analysis Regression, Generation Validation, Corpus Regression, Performance, Security, Fuzz) | Concept | Required test strategy coverage | cross-cutting (test infra) | Respective modules under test | All modules | Ongoing | Not Started | U | Yes | Process Contract | N/A (self-defining)
Architecture Tests | Concept | Enforce dependency direction / no theory leakage | cross-cutting (test infra) | Module Architecture (Sec 190–192) | CI gating | Ongoing | Implemented (partial) [v6] — every new analyzer this session (Cadence, Motif, Phrase Boundary, Period, Sentence, Form, Thematic Return Detection) shipped with its own architecture guard test confirming no upward/sideways layer leakage; full suite (157-161 tests across the session, confirmed passing at each branch) includes these | U | Yes | Process Contract | N/A (self-defining)

=== WORK / EDITION MODEL ===
Edition | Entity | Distinct published version of a Work | music-containers | Work | Source Variants | Post kernel | Not Started | B, F | Yes | Structural | Invariant
Arrangement | Entity | Adapted version of a Work | music-containers | Work | — | Post kernel | Not Started | B, F | Yes | Structural | Invariant
Revision | Entity | Version of a Work/Edition | music-containers | Work/Edition, Version | — | Post kernel | Not Started | A, B | Yes | Structural | Invariant
SourceVariant | Value | Variant reading / editorial disagreement | music-containers | Source | Scholarly Annotation | Post kernel | Not Started | B | Partial ("shall eventually support" — deferred normative) | Reference Concept (deferred) | TBD
ScholarlyAnnotation | Entity | Claim/target/author/citation/framework/evidence/uncertainty/status | music-interpretation | Annotation base | Analysis, Provenance | Post kernel | Not Started | I, K | Partial ("should support" — soft) | Reference Concept | TBD

=== OBSERVABILITY / REPRODUCIBILITY (cross-cutting) ===
Structured Logs / Traces / Metrics | Concept | Observability data, does not alter domain semantics | cross-cutting | — | Ops/monitoring | Ongoing | Not Started | U | Yes | Behavioral Contract | N/A (ops)
Correlation IDs / Analysis IDs / Generation IDs / Provenance IDs | Value | Cross-system tracing identifiers | cross-cutting | EntityID | Observability | Ongoing | Not Started | A, K | Yes | Structural | N/A (ops)
ReproducibilityRecord | Entity | activity_id, input_ids/versions, algorithm/framework version, config, seed, env, output_ids | music-provenance | ProvenanceActivity | Generation, Analysis reproducibility | Ongoing | Not Started | K | Yes | Structural | GenerationValidation, AnalysisRegression
Determinism Contract | Concept | States determinism guarantee or stochastic justification per algorithm | music-analysis / music-generation | Procedure/Generator definitions | Reproducibility | Ongoing | Not Started | K, U | Yes | Behavioral Contract | AnalysisRegression, GenerationValidation