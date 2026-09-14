# Registry — Structures, Semantics, Relationships, Provenance (L5–L6)

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13). Columns: Name | Category | Purpose |
Module | Depends On | Depended On By | Phase | Status | Spec Ref | Normative? | Contract
Type | Test Spec Ref. See `Registry_Index.md` for the full-Registry split and standing
verification warning.

## Structures / Occurrences

Motif | Structure | Abstract recurring pattern | music-structure | Event refs | MotifOccurrence, SimilarityResult | 5 | Implemented — plain L5 shell (id + EntityID-reference-list), detection logic deferred to L8 (Motif Detection). Framework-relative claims about a Motif instance use a generic Interpretation (L6) targeting it — no separate claim-shaped type | F | Yes | Structural | Relationship
MotifOccurrence | Occurrence | Situated instance of a motif | music-structure | Motif, region/events | Motif Analysis results | 5 | Implemented — references Motif via a bare EntityID (not RelationshipReference — confirmed correct as committed) | F | Yes | Structural | Relationship
Figure | Structure | Small structural unit | music-structure | Event refs | Phrase | 5 | Implemented (partial) — same L5 shell shape as Motif/Theme/Cadence | F | No (example only) | Reference Concept | TBD
Theme | Structure | Structural unit above phrase | music-structure | Phrase refs | Section | 5 | Implemented — L5 shell, consumed by Form Structure/Thematic Return Detection via ThemeOccurrence | F | No (example only) | Reference Concept | TBD
ThemeOccurrence | Occurrence | Situated instance of a theme | music-structure | Theme, region | Formal analysis | 5 | Implemented — Thematic Return Detection links ThemeOccurrence ids directly (not Section ids) via a "resembles" Relationship — the recurrence claim is about the material, not its container. Minimum-window-length floor added (PR #37) to fix over-matching | F | No (example only) | Reference Concept | TBD
Cadence | Structure | Formal closure unit | music-structure | Event refs, Harmonic context | Formal Analysis, Phrase Segmentation | 5/8 | Implemented — L5 shell, targeted by Cadence Detection/Classification's Interpretation output | F | Yes (explicit Phase 5/8 deliverable) | Structural | AnalysisRegression
PhraseOccurrence | Occurrence | Situated instance of a phrase | music-structure | Phrase, region | Formal Analysis | 5 | Implemented — constructed by Phrase Boundary Detection via a transient Phrase (L4) id-minting pattern, same shape MotifDetection uses for MotifOccurrence | F | No (naming-pattern example) | Reference Concept | TBD

**Durable pattern (do not re-litigate per branch):** L5 shell (the thing) + a generic L6
Interpretation targeting it (a framework-relative claim about it) — same relationship Chord
already has to Sonority. Where a claim spans two or more elements (Period, Sentence,
Form/Thematic Return), a Relationship (L6) links them and the Interpretation targets the
Relationship's id instead. See `Dependency_Layer_Specification.md` for the full history of
why this was briefly (and wrongly) changed, then reverted.

## Semantic Layer

Interval | Value | Directed/undirected/diatonic/chromatic distance | music-semantics | PitchIdentity/PitchClass pair | Sonority, Contour, Analysis | 3 | Not Started | D | Yes | Value Contract | Property-Based
Sonority | Value | Selected collection of simultaneous pitch-bearing events | music-semantics | Events, selection semantics | Chord Interpretation | 3 | Implemented — `SonorityBuilder` (PR #45) derives real Sonority objects from raw NoteEvents via attack-based beat-grid grouping (fixed quarter-note grid, a stated v1 default, not adaptive) | D | Yes | Value Contract | Golden
Chord | Interpretation | Framework-specific interpretation of a Sonority | music-interpretation | Sonority, FrameworkReference, Context | Harmonic Analysis | 8 | Implemented (minimal) — thin wrapper + real production via Chord Identification, including 4 seventh-chord qualities (dominant7, major7, minor7, half-diminished7 — fully-diminished7 deliberately excluded, requires harmonic minor, out of scope) alongside the original 7 diatonic triads, quality-keyed (root-relative interval set) not degree-keyed. EntityID uniqueness fix applied (PR #36, derived from sonority_target_id) | I, L | Yes | Behavioral Contract | Theory
Key | Interpretation | Contextual claim, not intrinsic to notes | music-interpretation | Context, FrameworkReference | Scale Degree, Roman Numeral Analysis | 8 | Implemented (minimal) — thin wrapper + real production via Key Estimation, major/natural-minor only | I, L | Yes | Behavioral Contract | Theory
Scale | Value | Ordered pitch collection under declared semantics | music-semantics | PitchClass, TuningSystem, Framework | Scale Degree, Scale Analysis | 3 | Not Started | D | No (attributes listed as "possible") | Reference Concept | TBD
ScaleDegree | Interpretation | Contextual mapping (pitch → degree) | music-interpretation | Scale, Context, FrameworkReference | Roman Numeral Analysis | 3/8 | Not Started | D, I | Yes | Structural | Theory
Contour | Value | Relative pitch change (up/down/same) | music-semantics | PitchIdentity sequence | Motif/Similarity Analysis | 3 | Implemented — direction is caller-supplied, not computed from PitchIdentity (deliberate — PitchIdentity is opaque, same constraint as Voice Leading). This makes Contour-based matching transposition-invariant by construction, which Motif Detection relies on for v1's exact-repetition/transposition-only scope | D | No (descriptive examples) | Reference Concept | TBD
RhythmicPattern | Value | Independently representable rhythm sequence | music-semantics | Duration/TimeSpan sequence | Pattern Matching, Motif Detection | 3 | Implemented — fully constructible/queryable from rhythm alone (no pitch header dependency) | C, D | Yes | Structural | Unit

## Relationship Model

PredicateDefinition | Entity | Metadata for a relationship predicate | music-relationship | — | Relationship | 6 | Implemented — confirmed via real Relationship usage | G | Yes | Structural | Relationship
Predicate Vocabulary (contains, precedes, derived_from, realizes, resembles, interpreted_as, etc.) | Value set | Initial extensible predicate list | music-relationship | PredicateDefinition | Relationship | 6 | Implemented (partial) — 3 of the documented vocabulary confirmed in real use: "precedes" (sequential order — Period antecedent→consequent, Form section order), "transformed_from" (derivation — Sentence statement→repetition→continuation, direction: derived-part→source), "resembles" (recurrence, no derivation direction claimed — Form thematic return). Extend this vocabulary before inventing new predicate names for a genuinely new relationship kind | G | No (explicitly extensible, not fixed) | Reference Concept | TBD

**Relationship (base type, L6, music-relationship) itself: Implemented** — confirmed via
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
