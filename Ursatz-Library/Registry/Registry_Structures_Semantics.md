# Registry — Structures, Semantics, Relationships, Provenance (L5–L6)

**Last verified against repo state:** 2026-10-05 (Ursatz `090695c`, PR #97). Columns: Name | Category | Purpose |
Module | Depends On | Depended On By | Phase | Status | Spec Ref | Normative? | Contract
Type | Test Spec Ref. See `Registry_Index.md` for the full-Registry split and standing
verification warning.

## Structures / Occurrences

Motif | Structure | Abstract recurring pattern | music-structure | Event refs | MotifOccurrence, SimilarityResult | 5 | Implemented (plain L5 shell, id plus EntityID-reference-list), detection logic deferred to L8 (Motif Detection). Framework-relative claims about a Motif instance use a generic Interpretation (L6) targeting it; no separate claim-shaped type | F | Yes | Structural | Relationship
MotifOccurrence | Occurrence | Situated instance of a motif | music-structure | Motif, region/events | Motif Analysis results | 5 | Implemented (references Motif via a bare EntityID, not RelationshipReference; confirmed correct as committed) | F | Yes | Structural | Relationship
Figure | Structure | Small structural unit | music-structure | Event refs | Phrase | 5 | Implemented (partial); same L5 shell shape as Motif/Theme/Cadence | F | No (example only) | Reference Concept | TBD
Theme | Structure | Structural unit above phrase | music-structure | Phrase refs | Section | 5 | Implemented (L5 shell), consumed by Form Structure/Thematic Return Detection via ThemeOccurrence | F | No (example only) | Reference Concept | TBD
ThemeOccurrence | Occurrence | Situated instance of a theme | music-structure | Theme, region | Formal analysis | 5 | Implemented. Thematic Return Detection links ThemeOccurrence ids directly (not Section ids) via a "resembles" Relationship, since the recurrence claim is about the material, not its container. Minimum-window-length floor added (PR #37) to fix over-matching | F | No (example only) | Reference Concept | TBD
Cadence | Structure | Formal closure unit | music-structure | Event refs, Harmonic context | Formal Analysis, Phrase Segmentation | 5/8 | Implemented (L5 shell), targeted by Cadence Detection/Classification's Interpretation output | F | Yes (explicit Phase 5/8 deliverable) | Structural | AnalysisRegression
PhraseOccurrence | Occurrence | Situated instance of a phrase | music-structure | Phrase, region | Formal Analysis | 5 | Implemented (constructed by Phrase Boundary Detection via a transient Phrase (L4) id-minting pattern), same shape MotifDetection uses for MotifOccurrence | F | No (naming-pattern example) | Reference Concept | TBD

**Durable pattern (do not re-litigate per branch):** L5 shell (the thing) plus a generic L6
Interpretation targeting it (a framework-relative claim about it); the same relationship Chord
already has to Sonority. Where a claim spans two or more elements (Period, Sentence,
Form/Thematic Return), a Relationship (L6) links them and the Interpretation targets the
Relationship's id instead. See `Dependency_Layer_Specification.md` for the full history of
why this was briefly (and wrongly) changed, then reverted.

## Semantic Layer

Interval | Value | Directed/undirected/diatonic/chromatic distance | music-semantics | PitchIdentity/PitchClass pair | Sonority, Contour, Analysis | 3 | Implemented (`src/semantics/interval.h`): semitone distance between PitchIdentity tokens, caller-supplied (computes nothing itself). Written-letter size is the separate GenericInterval (L2) | D | Yes | Value Contract | Property-Based
Sonority | Value | Selected collection of simultaneous pitch-bearing events | music-semantics | Events, selection semantics | Chord Interpretation | 3 | Implemented. `SonorityBuilder` (PR #45) derives real Sonority objects from raw NoteEvents via attack-based beat-grid grouping (fixed quarter-note grid, a stated v1 default, not adaptive) | D | Yes | Value Contract | Golden
Chord | Interpretation | Framework-specific interpretation of a Sonority | music-interpretation | Sonority, FrameworkReference, Context | Harmonic Analysis | 8 | Implemented (minimal); thin wrapper plus real production via Chord Identification, including 4 seventh-chord qualities (dominant7, major7, minor7, half-diminished7; fully-diminished7 deliberately excluded, requires harmonic minor, out of scope) alongside the original 7 diatonic triads, quality-keyed (root-relative interval set) not degree-keyed. EntityID uniqueness fix applied (PR #36, derived from sonority_target_id) | I, L | Yes | Behavioral Contract | Theory
Key | Interpretation | Contextual claim, not intrinsic to notes | music-interpretation | Context, FrameworkReference | Scale Degree, Roman Numeral Analysis | 8 | Implemented (minimal); thin wrapper plus real production via Key Estimation, major/natural-minor only | I, L | Yes | Behavioral Contract | Theory
Scale | Value | Ordered pitch collection under declared semantics | music-semantics | PitchClass, TuningSystem, Framework | Scale Degree, Scale Analysis | 3 | Implemented (`src/semantics/scale.h`, references a TuningSystem). Scale vocabulary (major, minor, modes) lives in `frameworks/standard-tonality` | D | No (attributes listed as "possible") | Reference Concept | TBD
ScaleDegree | Interpretation | Contextual mapping (pitch → degree) | music-interpretation | Scale, Context, FrameworkReference | Roman Numeral Analysis | 3/8 | Implemented (`src/interpretation/scale_degree.h`, requires a FrameworkReference); `standard_tonality_build_scale_degree` builds a populated one | D, I | Yes | Structural | Theory
Contour | Value | Relative pitch change (up/down/same) | music-semantics | PitchIdentity sequence | Motif/Similarity Analysis | 3 | Implemented. Direction is caller-supplied, not computed from PitchIdentity (deliberate, since PitchIdentity is opaque, same constraint as Voice Leading). This makes Contour-based matching transposition-invariant by construction, which Motif Detection relies on for v1's exact-repetition/transposition-only scope | D | No (descriptive examples) | Reference Concept | TBD
RhythmicPattern | Value | Independently representable rhythm sequence | music-semantics | Duration/TimeSpan sequence | Pattern Matching, Motif Detection | 3 | Implemented (fully constructible/queryable from rhythm alone, no pitch header dependency) | C, D | Yes | Structural | Unit
Texture | Interpretation | Framework-relative claim about a passage's texture (e.g. homorhythm) | music-interpretation | FrameworkReference, passage EntityID | standard-texture | 8 | Implemented (PR #64, `src/interpretation/texture.h`); requires a FrameworkReference | I | No | Structural | Theory

### Vocabulary enums (L6, `src/semantics/`)

Plain enums naming theory concepts. No EntityID, no data, no framework claim; meaning is the
using framework's convention. Admission rule (2026-10-05): a new enum enters the kernel only
when two or more frameworks need it. Existing single-framework enums stay until their framework
next changes, then move into it. Enums with no consumer are removed.

Enum | Used by | PR | Status
VoicePart | standard-voicing, standard-harmony, high-baroque-fugue | #65 | Implemented (shared)
BeatStrength | standard-counterpoint, standard-harmony, galant-schemas | #68 | Implemented (shared)
NonChordTone | standard-counterpoint, standard-harmony | #69-71, #83 | Implemented (shared)
ImitationAnswer | sixteenth-century-style, high-baroque-fugue | #74 | Implemented (shared)
CadenceType | standard-harmony, standard-form | #80 | Implemented (shared). Designated owner of cadence vocabulary; see Known_Gaps for the analysis-layer strings that predate it
MotionType, CounterpointSpecies | standard-counterpoint | #66-67 | Implemented (single framework; move on next change)
FugueRole | high-baroque-fugue | #75 | Implemented (single framework; move on next change)
OstinatoType, ImitativeRole | baroque-ground-bass | #76 | Implemented (single framework; move on next change)
SchemaFunction | galant-schemas | #77 | Implemented (single framework; move on next change)
ScaleDirection, SequenceKind | scale-harmonization | #79 | Implemented (single framework; move on next change)
HarmonicFunction, SeventhResolution, PhraseModelLabel, SixFourType, AppliedChordKind, ModulationKind, KeyChangeExtent | standard-harmony | #80-94 | Implemented (single framework; move on next change)
FormLevel, MotiveKind, PhraseRole, PhraseBeginning, PhraseEnding, IdeaLabel, SubphraseRole, ContinuationTrait, PhraseLevelForm | standard-form | #95-97 | Implemented (single framework; move on next change)
SubjectPart, SubjectCharacter, SchemaSense | none (tests only) | #75, #78 | Implemented, no consumer. Slated for removal

## Relationship Model

PredicateDefinition | Entity | Metadata for a relationship predicate | music-relationship | — | Relationship | 6 | Implemented (confirmed via real Relationship usage) | G | Yes | Structural | Relationship
Predicate Vocabulary (contains, precedes, derived_from, realizes, resembles, interpreted_as, etc.) | Value set | Initial extensible predicate list | music-relationship | PredicateDefinition | Relationship | 6 | Implemented (partial). 3 of the documented vocabulary confirmed in real use: "precedes" (sequential order, Period antecedent to consequent, Form section order), "transformed_from" (derivation, Sentence statement to repetition to continuation; direction derived-part to source), "resembles" (recurrence, no derivation direction claimed; Form thematic return). Extend this vocabulary before inventing new predicate names for a genuinely new relationship kind | G | No (explicitly extensible, not fixed) | Reference Concept | TBD

**Relationship (base type, L6, music-relationship) itself is Implemented.** Confirmed via
`src/relationship/relationship.h` (id/source/predicate/target/scope/context/qualifiers/
evidence/uncertainty/provenance) and real `relationship_create()` calls in
`period_detection.c`, `sentence_detection.c`, `thematic_return_detection.c`,
`form_structure_detection.c`. First real use of Relationship from the analysis layer was
Period Detection.

## Provenance

ProvenanceActivity | Entity (PROV) | Process producing/changing info (import, analysis, transform, generation) | music-provenance | Agent | Derivation | 6 | Not Started | K | Yes | Structural | Provenance
ProvenanceAgent | Entity (PROV) | Human/software/org/ML model/external source | music-provenance | — | Activity, Derivation | 6 | Not Started | K | Yes | Structural | Provenance
Derivation | Entity (PROV) | Output/input/activity/params/agent/version record | music-provenance | Activity, Agent | Provenance Graph | 6 | Not Started | K | Yes | Structural | Provenance
ProvenanceGraph | Concept | Acyclic lineage graph | music-provenance | Derivation(s) | Audit/Reproducibility | 6 | Not Started | K | Yes | Invariant | Provenance
AuditEvent | Entity | Who changed what/when/why (distinct from provenance) | music-provenance | Agent, timestamp | Audit History | 6 | Not Started | K | Yes | Structural | Provenance
Annotation | Entity | Human-authored analysis/claim | music-interpretation | Author, Target, Context | Promotion to authored content | 4/6 | Not Started | I, K | Yes | Structural | Context

**Note:** only `ProvenanceReference` (the opaque L0–L1 pointer, see `Registry_Foundations.md`)
is implemented. The full Provenance/Activity/Agent/Derivation model above remains entirely
unbuilt — this is a real gap, not a documentation lag, and blocks any real provenance lineage
view (see `Overview/Apps_Roadmap.md`'s Phase 2 remaining work).
