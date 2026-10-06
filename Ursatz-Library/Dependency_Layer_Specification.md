# Dependency & Layer Specification

**Last verified against repo state:** 2026-10-05 (Ursatz `090695c`, PR #97). Layer contents
table updated; three rules added on 2026-10-05 (framework extension modules, vocabulary enums,
notation values at L2), see "Framework extension modules" below.

Source: Technical Design Document, System Registry.

## Status

Both long-standing open issues remain resolved:
- **Interpretation-family → Framework ordering**, resolved via `FrameworkReference` (L6
  holds a reference, resolution happens at L7+).
- **Theory → Corpus ordering**, confirmed already correctly handled. Theory-layer
  statistical objects (Tendency, StatisticalModel, StyleModel) consume Corpus data only as
  opaque pre-computed values, never importing the Corpus module itself.

**Standing correction (2026-09-09, kept for the record):** an earlier revision of this
document recorded a "Framework Output Contract decision" relocating Motif, Cadence, Theme,
and PhraseOccurrence from L5 (music-structure) to L6 (music-interpretation), reasoning that
their intended fields (FrameworkReference, Context, Evidence) were L6 types. That decision
was made without checking actual repo state first, and was **wrong**: all four types already
existed, committed, as plain L5 shells (id + EntityID-reference-list, detection logic
deliberately deferred to L8). There was no forward dependency to resolve. This is reverted in
full; see Layer contents below, and `Overview/Master_Roadmap.md`'s process discipline
section for the lesson this taught.

The **actual** resolution, confirmed by building against it across the full Framework-stage
branch sequence (Cadence Detection, Motif Detection, Phrase Boundary Detection, Period
Detection, Sentence Detection, Form Structure Detection, Thematic Return Detection, all
merged and verified in code as of 2026-09-13): a framework-relative claim about an L5 shell
is expressed as the existing generic `Interpretation` type (L6, the same type Chord/Key
already use), with its `target` set to the L5 shell's EntityID. No new named Interpretation-
family type was needed for any of these. Where a claim links *two or more* L5/L6 elements
(e.g. antecedent+consequent, statement+repetition+continuation, a section boundary, a
thematic return), the mechanism is a `Relationship` (L6, music-relationship) linking them,
with the `Interpretation` targeting the Relationship's id instead of a single element. This
two-tier pattern is the standing default for any future Framework-stage work.

**Relationship (L6, music-relationship): confirmed implemented and in real use, verified
2026-09-13.** Predicates actually in use in code: `precedes` (sequential order; Period
antecedent→consequent, Form section order), `transformed_from` (derivation; Sentence
statement→repetition→continuation, direction: derived-part → source), `resembles`
(recurrence with no claimed derivation direction; Form thematic return, Theme-to-Theme).

**Relationship persistence: no repository home yet, stopgap confirmed still in use
(2026-09-13).** `RelationshipRepository` does not exist in Ursatz's persistence layer.
Ursatz-GUI's Framework-analyzer wiring needed to persist real Relationship instances
(Period/Sentence/Form/Thematic Return output) and could not wait on this, so it added an
Ursatz-GUI-owned SQLite table (`ursatz_gui_relationships`) mirroring its existing index/meta
table pattern. This is application-layer, outside the numbered stack, same as any other
Persistence module. It is not a violation of layer rules, but also not the eventual Ursatz-side
`RelationshipRepository`. Retire that table in favor of it once/if it's built.

This specification is the working dependency contract.

## Layer order

```
L0  Identity
L1  Core
L2  Time / Pitch
L3  Events
L4  Containers
L5  Structures
L6  Context / Semantics
L7  Theory
L8  Analysis / Generation / Transformation
L9  I/O / Applications
```

Cross-cutting (attach to any layer, dependency direction governed by their own ceiling, not
layer order): Provenance, Validation, Security, Observability, Persistence.

## Dependency rule

A module in layer **N** may depend only on modules in layers **0..N**, plus any cross-cutting
module it's permitted to depend on. A module in layer **N** shall never depend on any module
in layers **N+1..9**.

- A numbered layer MAY depend on a cross-cutting module, subject to that module's individual
  ceiling (below); the per-module ceiling governs over this general permission.
- A cross-cutting module MUST obey its own stated ceiling and MUST NOT depend on any numbered
  layer above it. "Cross-cutting" describes *who may use it*, not *what it may use*.
- Persistence is outside the numbered domain stack entirely, I/O-adjacent, and it is not "layer 9,"
  nor exempt from restriction.

## Layer contents (module → layer), current implementation status

| Layer | Modules | Status |
|---|---|---|
| L0 Identity | music-identity (EntityID, Version, VersionModel, SchemaVersion) | Implemented |
| L1 Core | music-core (Entity, Value, Occurrence, five equality notions, reference types, Canonical Musical Model/IR) | Implemented. Canonical IR is the emergent graph, not a module |
| L2 Time/Pitch | music-time (Duration, MusicalTime, TimeSpan, TimeInstant, TemporalAnchor, TimeSignature, NoteValue, Tuplet, MetronomeMarking, HistoricalPeriod), music-pitch (PitchIdentity, PitchSpelling, PitchClass, PitchNumericValue, TuningSystem, PitchRealization, WrittenPitch, SoundingPitch, Clef, StaffPosition, Accidental, KeySignature, GenericInterval) | Implemented. MeterMap, TempoMap, PitchTrajectory not started |
| L3 Events | music-events (Event, NoteEvent, RestEvent, ControlEvent, Tie, Slur, Articulation, Dynamics, TempoMarking, StructuralMarker, Instrument*, InstrumentCapability*) | Implemented |
| L4 Containers | music-containers (Container, Voice, Layer, Staff, Part, Measure, Idea, Subphrase, Phrase-container, Section, Movement, Work, Edition, Arrangement, Revision, SourceVariant) | Implemented. Voice assignment real (interval-graph coloring); Measure's MeterMap reference is a bare EntityID |
| L5 Structures | music-structure (Structure, Motif, MotifOccurrence, Figure, Theme, ThemeOccurrence, Cadence, PhraseOccurrence) | All implemented as plain shells |
| L6 Context/Semantics | music-context (Context), music-semantics (Interval, Sonority, SonorityBuilder, Scale, Contour, RhythmicPattern, 32 vocabulary enums), music-relationship, music-interpretation (Interpretation, Assertion, Observation, Hypothesis, Evidence, Uncertainty, Chord, Key, ScaleDegree, Texture) | Implemented. Uncertainty has 3 of 8 kinds |
| L7 Theory | music-theory, music-rules (Framework, Rule, RuleEvaluation, Constraint, Preference, Heuristic, Tendency, StatisticalModel, StyleModel) | Implemented as base types. `preference_rank` is a deliberate stub (`constraint_evaluate` is real, via framework-registered callbacks); Tendency/StatisticalModel/StyleModel are shape only |
| Framework extensions | `frameworks/<name>/`, 17 modules (see `Registry/Registry_Theory_Analysis.md`) | Implemented as rule libraries over caller-supplied data; only `common-practice-minimal` is used by analysis |
| L8 Analysis/Generation/Transformation | music-analysis (14 analyzers + FeatureExtraction), music-generation, music-transform | Analysis implemented; Transform: 12 concrete transforms; Generation: base types implemented, Generator/IntentCompiler orchestration not started |
| L9 I/O/Applications | music-io (MIDI import, MIDI key-signature scan), music-export (MusicXML), music-persistence, music-corpus, corpus_stats, music-notation, music-performance, Query interface, Applications | MIDI import, key-signature scan, MusicXML export, persistence (minimal), corpus, corpus_stats (minimal) implemented; notation, performance, query not started |

`*` = provisional placement, unresolved boundary question (see Domain Specification).

## Cross-cutting modules

| Module | Dependency ceiling | May be depended on by | Notes |
|---|---|---|---|
| Provenance | L0–L1 only | Any layer needing lineage (L1+) | MUST NOT depend on any layer above Core. No layer's own semantics may depend on Provenance to define correctness — it records lineage, not domain state. |
| Observability | L0 (EntityID) only | Any layer | MUST NOT alter domain semantics. |
| Security (plugin contract, sandboxing, query safety limits) | L0–L1 only | L7–L9 (plugins, queries) | MUST NOT be bypassed by any layer it governs. |
| Validation | Own layer + all lower layers | All layers | A validator for layer N depends on layer N plus everything N may depend on. MUST NOT depend upward. |
| Persistence (repositories, Cache) | Canonical IR / L1 only | I/O and application code outside the numbered stack | Outside the numbered stack. MUST NEVER be imported by L0–L8. See Relationship-persistence stopgap note above. |

No cross-cutting module may become an escape hatch around the layer architecture; routing
through one doesn't exempt a dependency from the rules above.

**Provisional layer placements** (kept as-is, flagged as dependency-magnet risk):
- **Instrument/InstrumentCapability (L3)**, referenced by Events, Performance, Generation
  constraints, and Notation; a plausible candidate for its own boundary once those
  relationships are fully specified.

**music-corpus (L9): placement resolved, no longer provisional.** `corpus` (L9) depends
only on `identity` (L0) and `persistence` (L9), dependency-pure and sibling to persistence.
Analyzer and Generator depend on `corpus` directly as peers; neither routes through the
other. `corpus_stats` (L9-adjacent) is a separate module depending on `corpus`, `persistence`,
`interpretation`, and `identity`; this split exists because chord-level statistics require
reading `interpretation` data, which `corpus` itself must not depend on.

## Explicit prohibitions

**Layer leakage:**
```
Core MUST NOT import Theory | Pitch MUST NOT import Harmony/any Framework
Time MUST NOT import Notation
Events / Containers MUST NOT import Structures, Context, Theory, Analysis, I/O
Domain (L0–L8) MUST NOT import Persistence or SQL/storage tech
Plugins MUST NOT bypass capability/security boundaries
```

**Canonical IR authority:** the Canonical Musical Model/IR (L1) is the authoritative
theory-neutral representation. Derived layers (L5–L9) may interpret, analyze, transform,
render, project, or generate from it, but MUST NOT silently redefine canonical semantics:
```
Theory MUST NOT mutate Canonical Events
Notation / Performance MUST NOT redefine Canonical Events
```

**Semantic-leakage prohibitions:**
```
Analysis MUST NOT introduce theory-specific semantics into the Canonical IR
Generation MUST NOT require a specific Theory Framework unless explicitly selected
  by the GenerationPlan or equivalent configuration
Query projections MUST NOT become authoritative domain state
```

**Pitch numeric semantics:**
```
L5-L8 analyzers MUST NOT compute real interval/tuning values (e.g. "is this a perfect
  fifth", semitone distance) directly from PitchIdentity — it is deliberately opaque
  (Law 18). Such computation is blocked on TuningSystem/PitchRealization at the kernel
  level unless a consumer application does its own application-level math outside the
  kernel (e.g. Ursatz-Analyzer's pitch_class_util does its own 12-TET-specific math for
  Chord/Key pipelines as an application-level concern, not a kernel one).
```
**Current real status (verified 2026-10-05):** the numeric path exists. `PitchNumericValue`
(L2) is an exact-rational pitch position under a `TuningSystem`; MIDIImporter produces one per
NoteEvent under a "MIDI note number" tuning, and Transposition, Inversion and
IntervallicManipulation consume it. Voice Leading, Cadence Classification's PAC/IAC
distinction, and Motif Detection still take caller-supplied or Contour/RhythmicPattern-derived
data and compute nothing from pitch themselves. Frameworks compute intervals from
`PitchSpelling` (letter, accidental, octave), which is written-pitch arithmetic and allowed.
See `Status.md` and `Known_Gaps.md`.

## Registry-first dependency rule

The System Registry is an architectural authority, not documentation:

> Every inter-module dependency MUST be represented in the Registry before implementation.
> Architecture tests SHALL reject dependencies not authorized by the Registry.

Undocumented sibling coupling is prohibited at any layer, including within the same layer. A
needed dependency not yet in the Registry must be reviewed and added there first, never
introduced informally in code and reconciled later.

**Enforcement, as built:** each module and framework has its own architecture test with a
hand-kept allowlist of the directories it may include. There is no single check against the
Registry. Kernel guards run in CI; framework guards do not yet (see `Known_Gaps.md`). Keeping
an allowlist and its Registry row in step is a manual review item on every PR that adds an
include edge.

**The 2026-10-02 to 10-04 lapse.** Ursatz PRs #49 to #97 added 16 frameworks, 48 kernel
headers and 6 framework-to-framework edges without Registry entries first. They were reviewed
and back-filled on 2026-10-05 (see `Reconciliation_Log.md`). The rule stands unchanged.

**Verify before deciding, not just before implementing:** an architecture-level placement
decision is only as good as the repo-state check behind it. Confirm actual committed state
before making an architecture decision, not only before a branch starts implementing one.
This is the standing lesson from the 2026-09-09 L5/L6 misplacement above, and it now also
applies to documentation claims generally; see `Reconciliation_Log.md`.

## Framework extension modules (decided 2026-10-05)

- **Placement.** Concrete theory lives in `frameworks/<name>/`, outside the numbered stack,
  built on L7's `Framework`/`Rule`/`Constraint` without modifying them. No `src/` module may
  include a framework header, except `analysis`'s ChordIdentification/KeyEstimation, which
  compile into the `common_practice_minimal` target.
- **Ceiling.** L0 to L6, plus `theory/framework.h`, `rules/rule.h`, `rules/constraint.h`.
- **Framework-to-framework edges** are allowed when recorded in the Registry. Current edges:
  `sixteenth-century-style`, `high-baroque-fugue`, `baroque-ground-bass`, `galant-schemas`,
  `scale-harmonization` on `standard-counterpoint`; `scale-harmonization` on `galant-schemas`.
  Frameworks are "independently selectable," not necessarily independent.
- **Perception path rule.** A new framework, or a new rule in an existing one, is not started
  until the analysis layer can produce that rule's inputs from imported music, or the same
  branch builds that path. Rule libraries without a path from MIDI/notation are not added.

## Vocabulary enums at L6 (decided 2026-10-05)

`src/semantics/` may hold plain enums naming theory concepts (`CadenceType`, `VoicePart`, ...):
no EntityID, no data, no framework claim, only names and name-level predicates. This is a
narrow exception to the Technical Design Document's kernel admission criteria, granted because
such a name carries no theory by itself. Conditions:

- A new enum enters the kernel only when two or more frameworks need it.
- A single-framework enum lives in that framework. The 24 existing single-framework enums move
  into their framework the next time that framework changes.
- Enums with no consumer are removed (`SubjectPart`, `SubjectCharacter`, `SchemaSense`).

## Notation values at L2 (decided 2026-10-05)

`Clef`, `StaffPosition`, `ClefResolution`, `Accidental`, `KeySignature` (pitch) and
`TimeSignature`, `NoteValue`, `Tuplet`, `MetronomeMarking` (time) are accepted at L2 as plain,
theory-free values. They describe written symbols, not musical reality (Law 3 holds: nothing
treats them as authoritative over NoteEvents), and lower layers and importers need them
(MIDIImporter must produce `TimeSignature` and `KeySignature`). `music-notation` (L9) remains
the home for layout, engraving and rendering. `HistoricalPeriod` is accepted in `time/` as a
plain value.

## L9 detail

L9 is external/application-facing. Being in L9 doesn't make any of these authoritative over
the canonical domain model (L1):

| Sub-area | Modules | Role |
|---|---|---|
| I/O adapters | music-io (MIDI import, MIDI key-signature scan), music-export (MusicXML); MusicXML/MEI/ABC import, other renderers, AudioAnalyzer, OCRImporter, LossModel not started | Converts between external formats and Canonical IR. Never defines domain semantics. |
| Notation/rendering | music-notation | Not started at the kernel level (Ursatz-GUI does its own OSMD-based rendering from exported MusicXML — an application concern, not this module). |
| Performance realization | music-performance | Not started. |
| Corpus/statistical infrastructure | music-corpus, corpus_stats | Implemented (minimal). Derived, non-authoritative statistical knowledge. |
| Query interfaces | Query Interface, PatternMatching, SimilarityModel, SearchResult, Views | Not built. Framework-stage analyzer output is structurally queryable by construction (real Relationship predicates, structured Interpretation claims) but nothing queries it yet. |
| Applications | Composition, Practice, Ear Training, Analysis, Corpus, OCR, Audio, Notation, Performance apps | Ursatz-Analyzer and Ursatz-GUI are the two real consumers today. |

## Architecture test mapping

| Test | Enforces |
|---|---|
| core cannot import theory | L1 ⇏ L7 |
| pitch cannot import harmony/frameworks | L2 ⇏ L7 |
| time cannot import notation | L2 ⇏ L9 |
| domain cannot import persistence | L0–L8 ⇏ Persistence |
| domain cannot import SQL/storage tech | L0–L8 ⇏ storage tech |
| theory cannot mutate canonical events | L7 write-access to L1–L3 = forbidden |
| notation/performance cannot redefine canonical events | L9 write/redefine access to L1–L3 = forbidden |
| analysis cannot introduce theory-specific semantics into Canonical IR | L8 writes to L1 restricted to theory-neutral fields |
| query projections cannot become authoritative domain state | Query/View modules read/derive only |
| plugins cannot bypass capability/security boundaries | L7–L9 plugin code ⇏ Security bypass |
| undocumented inter-module dependencies rejected | Any include outside a module's allowlist fails its architecture test (allowlists kept in step with the Registry by review) |

Per Ursatz's own architecture-guard tests (regex-scanning `#include` lines against an
allowlist, one per module), the core layer-leakage checks above are implemented and passing
in CI for every kernel module. Every framework also has its own guard, but framework tests do
not run from the root build or CI yet, and `architecture_common_practice_minimal` fails when
run (its allowlist omits its own seventh-chord headers). See `Known_Gaps.md`.

## Resolved issues

### Interpretation-family ↔ Framework ordering — RESOLVED
Previously listed `Interpretation`/`Chord`/`Key`/`ScaleDegree` (L6) as depending directly on
`Framework` (L7), a forward dependency disallowed under strict layer order. Resolved via
**option 1**: these types depend only on a `FrameworkReference` (ID/version) living at L6;
resolution to the actual Framework happens at L7+.

### Theory (L7) ↔ Corpus (L9) ordering — RESOLVED
Previously listed `Tendency`/`StatisticalModel`/`StyleModel` (L7) as depending on
`Corpus`/`Corpus Statistics` (L9), also a forward dependency. Confirmed resolved in
practice, as Theory-layer statistical objects consume corpus data only as opaque pre-computed
values, never importing the Corpus module itself.

### L5 Structure-family placement — RESOLVED (corrected 2026-09-09 error)
See Status above. Motif, MotifOccurrence, Cadence, Theme, ThemeOccurrence, PhraseOccurrence,
Figure all confirmed at L5 as originally committed. No forward dependency existed.

## Normative language key

- **MUST/SHALL** — mandatory architectural requirement.
- **SHOULD** — recommendation, deviation requires justification.
- **MAY** — permitted, not required.
- **provisional** — placement not yet frozen; may change.

This document does not elevate illustrative Registry examples to new normative requirements
beyond what the Registry states.
