\# Ursatz Reconciliation Report (Read-Only Audit)

&#x20;

Repo: `/home/user/Ursatz`, HEAD `8768d8e` ("Merge pull request #11 ... feat/transform-analysis-generation").

142 commits total, one PR-merge per layer/module pass (`identity` -> `core` -> `time/pitch` -> `events` -> `containers` -> `structure` -> `context/semantics/relationship/interpretation` -> `theory/rules` -> `transform/analysis/generation`).

&#x20;

\*\*Headline finding\*\*: this codebase is unusually self-documenting. Nearly every deferral, deviation, and placeholder is called out in a doc-comment at the point of the shortcut, and `docs/ARCHITECTURE.md` (665 lines) mirrors that at the module level. I verified its claims by direct grep/read rather than trusting it, and found it accurate everywhere I checked. The genuinely "silent" items are a small, specific list — see the final section.

&#x20;

No code was modified during this audit.

&#x20;

\---

&#x20;

\## L0 — Identity (`src/identity`, music-identity)

&#x20;

Files: `entity\_id.{h,c}`, `version.{h,c}`, `version\_model.{h,c}`, `schema\_version.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*: `EntityID` (opaque, `entity\_id\_create`/`\_destroy`/`\_equals`/`\_value`, no mutator — `src/identity/entity\_id.h:8-16`), `Version`, `VersionModel`, `SchemaVersion` with round-trip/serialization tests (`tests/identity/test\_version.c`, `test\_schema\_version.c`).

2\. \*\*Deferred\*\*: nothing flagged at this layer beyond migration mechanics being a "placeholder" test only (`066b16f feat - Add SchemaVersion serialization and migration-placeholder tests`).

3\. \*\*Deviation\*\*: none found.

4\. \*\*Silent decisions\*\*: none found. `EntityID` has no mutator function anywhere in `src/identity` (`grep \_set\_|\_mutate|\_modify` = empty) — immutability holds.

\## L1 — Core (`src/core`, music-core)

&#x20;

Files: `entity.{h,c}`, `value.{h,c}`, `occurrence.{h,c}`, `equality.h`, `voice\_reference.{h,c}`, `relationship\_reference.{h,c}`, `provenance\_reference.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*: `Entity` (identity equality via EntityID+SchemaVersion, `src/core/entity.h:29-53`), `Value` (content equality), `Occurrence`, the five equality kinds (`equality.h`: Identity/Value/Structural/Semantic-contract/Canonical).

2\. \*\*Deferred, with stated reason\*\* (`docs/ARCHITECTURE.md:117-133`, mirrored in `src/core/entity.h:16-25`):

&#x20;  - `Entity.provenance\_reference` field from the Registry is \*\*not wired into `Entity`\*\* even though `ProvenanceReference` exists as a type. Comment: "ProvenanceReference is a proposed L1 reference type not yet added to the Registry ... deferred until that is resolved" (`src/core/entity.h:20-25`).

&#x20;  - `SemanticEquality` is a function-pointer contract only (`SemanticEqualsFn`), no real implementation — needs L6 `Context`.

&#x20;  - "Canonical IR" is documented as \*not a module to import\* — it's an emergent graph, nothing to build here.

3\. \*\*Deviation — flagged explicitly in the task brief, worth restating with evidence\*\*: The Domain Spec is described as marking `VoiceReference`, `RelationshipReference`, and `ProvenanceReference` "PROPOSED ONLY, NOT YET IN REGISTRY, MUST NOT be implemented until Registry synchronization." The repo nonetheless implements all three in commit `bf00705` ("feat - Add VoiceReference, RelationshipReference, and ProvenanceReference", authored 2026-09-03, same PR as the rest of L1's `music-core` pass). I searched `docs/ARCHITECTURE.md` and every commit message across the repo for the phrase "Registry sync\[hronization]" / "proposed only" / "must not be implemented" and found \*\*no occurrence anywhere\*\* — the specific blocking language from the Domain Spec is not acknowledged or reasoned about in-repo. The types themselves are thoroughly documented (opaque EntityID-shaped wrappers, three-state presence model for `ProvenanceReference` — `src/core/provenance\_reference.h:8-13`), but the act of proceeding \*despite\* the Registry-sync blocker is undocumented. This is the single clearest case in the repo of implementation outrunning a stated spec gate — reported per your instruction, not adjudicated.

4\. \*\*Silent decisions\*\*: none beyond #3 above (which is disclosed as a fact but not as "we are aware this defies a blocking clause").

\## L2 — Time / Pitch (`src/time`, `src/pitch`)

&#x20;

Files: `duration.{h,c}`, `musical\_time.{h,c}`, `time\_span.{h,c}`, `time\_instant.{h,c}`, `temporal\_anchor.{h,c}`; `pitch\_identity.{h,c}`, `pitch\_spelling.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd, verified directly\*\*:

&#x20;  - Law 19 (no float/double for rationals): `grep -rn "float\\|double" src/time src/pitch` returns \*\*zero code hits\*\*, only doc-comments citing "Law 19" (`src/time/musical\_time.h:9-10`, `duration.h:8`, `time\_instant.h:12`). All four time types use numerator/denominator ints reduced to lowest terms.

&#x20;  - `TimeSpan` requires explicit `boundary` at construction, no default: `time\_span\_create(..., TimeSpanBoundary boundary, ...)` with `TIME\_SPAN\_ERR\_INVALID\_BOUNDARY` (`src/time/time\_span.h:34-39`); comment explicitly instructs deserializers to reject an omitted boundary rather than assume CLOSED.

&#x20;  - `PitchIdentity`'s token is \*\*not\*\* EntityID-typed: constructor takes `(const void \*token, size\_t token\_size, ...)` — no `identity/entity\_id.h` include in `pitch\_identity.h`, so an `EntityID\*` argument would not compile (`src/pitch/pitch\_identity.h:1-32`).

&#x20;  - No mutators anywhere in `src/time` or `src/pitch`.

2\. \*\*Deferred\*\*: `PitchClass`, `PitchRealization`, `TuningSystem`, `WrittenPitch`, `SoundingPitch` explicitly named as "Phase 2/3 (Deliverable D)" and out of scope (`docs/ARCHITECTURE.md:188-190`). `TimeSpan`'s boundary vocabulary flagged PROVISIONAL per Domain Spec §16.3 (`time\_span.h:11-14`).

3\. \*\*Deviation\*\*: `PitchIdentity` equality is admitted to be a placeholder (raw byte equality), not the "true equivalence relation," pending Deliverable D (`pitch\_identity.h:24-27`).

4\. \*\*Silent decisions\*\*: none found — every shortcut here carries an explicit comment.

\## L3 — Events (`src/events`)

&#x20;

Files: `event.{h,c}`, `note\_event.{h,c}`, `rest\_event.{h,c}`, `control\_event.{h,c}`, `tie.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*:

&#x20;  - `NoteEvent.optional\_voice` is `VoiceReference\*` (L1 opaque ref), never a resolved `Voice`. Verified: `grep -rn "#include" src/events` shows \*\*no `containers/` include anywhere\*\* (only `core/`, `identity/`, `pitch/`, `time/` includes — see `src/events/note\_event.h:6-10`).

&#x20;  - `tests/events/test\_architecture\_events.c` independently enforces this (allowlist scan, see L6 example test below for the identical pattern); it passed in the actual test run (`ctest -R architecture\_events` → Passed).

&#x20;  - `ProvenanceReference` three-state fail-fast at construction: `note\_event\_create` returns `NOTE\_EVENT\_ERR\_NULL\_PROVENANCE` if `provenance == NULL` — verified in `src/events/note\_event.c:31` (`if (provenance == NULL) { ... }`), so a caller cannot silently omit provenance.

&#x20;  - Two distinct equality functions: `note\_event\_equals\_by\_id` vs `note\_event\_equals\_by\_value` (`note\_event.h:95-107`).

&#x20;  - No mutator on `NoteEvent`, `RestEvent`, `ControlEvent`, or `Tie`.

2\. \*\*Deferred, stated reasons\*\* (`docs/ARCHITECTURE.md:235-243`, `note\_event.h:16-22`):

&#x20;  - `Slur`, `Articulation`, `Dynamics` (Registry Phase 2) — not built.

&#x20;  - `Instrument`, `InstrumentCapability` (Registry Phase "1+") — skipped, cited as "provisional placement, property list explicitly flagged illustrative/non-normative."

&#x20;  - `NoteEvent`'s Entity/Event/Occurrence category is UNRESOLVED upstream (Domain Spec §9, §16.1); `NoteEvent` deliberately does NOT build on `core/entity.h`'s `Entity` to avoid pre-judging that boundary.

3\. \*\*Deviation\*\*: none beyond the category-boundary non-resolution (which is a documented deferral, not really a deviation).

4\. \*\*Silent decisions\*\*: none found.

\## L4 — Containers (`src/containers`)

&#x20;

Files: `container.h` (no `.c` — pure marker type, no fields, no functions), `voice.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*: `Container` (empty base marker, `src/containers/container.h:25` — `typedef struct Container Container;` with no accessors at all), `Voice` (schema `{id}` only, per Domain Spec §10 — `src/containers/voice.h:9-12`). Identity equality only; no mutator (Law C2, `voice.h:22-23`).

2\. \*\*Deferred, extensively and explicitly\*\* (`docs/ARCHITECTURE.md:271-275`, `container.h:5-18`): `Layer`, `Staff`, `Part`, `Measure` (Registry Phase 2); `Phrase` (container use), `Section`, `Movement`, `Work`, `Edition`, `Arrangement`, `Revision`, `SourceVariant` (Registry Phase 5, several marked "Reference Concept"/non-normative). This is a \*large\* fraction of the L4 module spec left unbuilt — worth flagging to the user even though it is well-documented, since the module's actual surface area (2 files) is far smaller than the Dependency Spec's L4 type list implies.

3\. \*\*Deviation\*\*: none — the gap is disclosed as scope, not silently dropped.

4\. \*\*Silent decisions\*\*: none found. Voice's lack of a membership/event list is explicitly called out as intentional (`voice.h:10-13`), not an oversight.

\## L5 — Structure (`src/structure`)

&#x20;

Files: `structure.{h,c}`, `motif.{h,c}`, `motif\_occurrence.{h,c}`, `cadence.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*: `Structure` (real base type composing `Entity` + non-duplicating `EntityID` event refs — `src/structure/structure.h:14-19`), `Motif`, `MotifOccurrence` (the one type that embeds `Occurrence` directly, per `docs/ARCHITECTURE.md:303-307`), `Cadence` (Phase-5 shell, `optional\_harmonic\_context` as opaque nullable `Value`, `cadence.h`).

2\. \*\*Deferred, stated reasons\*\*: `Figure`; `Theme`/`ThemeOccurrence`; `PhraseOccurrence` — Registry Normative = No / "example only" (`docs/ARCHITECTURE.md:321-324`). `Section`, `Movement`, `Work` explicitly deferred to L4 container work (never built there either — see L4 above), and Cadence/Motif \*detection\* logic deferred to L8 `music-analysis` (built there — see L8 below).

3\. \*\*Deviation\*\*: none.

4\. \*\*Silent decisions\*\*: none found.

\## L6 — Context / Semantics / Relationship / Interpretation (`src/context`, `src/semantics`, `src/relationship`, `src/interpretation`)

&#x20;

Files: `context.{h,c}`; `interval.{h,c}`, `sonority.{h,c}`, `rhythmic\_pattern.{h,c}`; `relationship.{h,c}`, `predicate\_definition.{h,c}`; `uncertainty.{h,c}`, `evidence.{h,c}`, `assertion.{h,c}`, `observation.{h,c}`, `hypothesis.{h,c}`, `interpretation.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*:

&#x20;  - No sideways L6 imports: direct grep of all four modules' `#include` lines (`src/interpretation`, `src/context`, `src/semantics`, `src/relationship`) shows only stdlib headers plus `core/`, `identity/`, `time/`, `pitch/` (none reference each other). Every cross-module reference (Context, Evidence, Relationship targets) goes through a bare `EntityID`, e.g. `relationship.h`'s `context\_id`/`evidence\_id` fields (`src/relationship/relationship.h:27-37`).

&#x20;  - `Uncertainty` implemented as a 3-variant tagged type (`UNKNOWN`, `SCALAR\_CONFIDENCE`, `CANDIDATE\_SET`) with exact-rational fields, not a scalar (`src/interpretation/uncertainty.h`).

&#x20;  - `Evidence` keeps strength/relevance/reliability as three separate rational fields, never collapsed to one score.

&#x20;  - `Hypothesis` supports a `HypothesisCandidate` array (`candidate\_count >= 1`), not a single value+confidence.

&#x20;  - `Interpretation.target\_id` is an opaque `EntityID` with no accessor resolving it to a mutable object (Law 8), and multiple competing Interpretations on the same target are legal by construction.

2\. \*\*Deferred, stated reasons\*\* (`docs/ARCHITECTURE.md:427-435`): `Scale` (non-normative, reference only), `ScaleDegree` (needs Framework, Phase 3/8), `Contour` (non-normative), `Chord` and `Key` (Phase 8, depend on Framework — \*\*not implemented anywhere in the repo\*\*; there is no `chord.h`/`key.h` under `src/interpretation`), `Annotation`/`ScholarlyAnnotation` ("post kernel" soft requirement), `PitchClass`/`PitchRealization`/`TuningSystem` (Phase 3 Deliverable D).

3\. \*\*Deviation / OPEN ISSUE §7.1 — verified directly, not resolved (see dedicated section below)\*\*.

4\. \*\*Silent decisions\*\*: none beyond §7.1's resolution method itself, which is disclosed in a large comment block at the top of `interpretation.h` (`src/interpretation/interpretation.h:13-24`) — so even that is not "silent," just an open item correctly carried forward.

\## L7 — Theory / Rules (`src/theory`, `src/rules`)

&#x20;

Files: `framework.{h,c}`, `heuristic.{h,c}`, `tendency.{h,c}`, `statistical\_model.{h,c}`, `style\_model.{h,c}`; `rule.{h,c}`, `constraint.{h,c}`, `preference.{h,c}`.

&#x20;

1\. \*\*Implemented as spec'd\*\*:

&#x20;  - `rules` legitimately depends on `theory`: `src/rules/rule.h:8`, `constraint.h:8`, `preference.h:8` all `#include "theory/framework.h"` — this is the one \*documented, allowed\* L7 sibling dependency (Registry: `Rule`/`Constraint`/`Preference` each hold an unowned `Framework` reference), confirmed by `tests/rules/test\_architecture\_rules.c`'s allowlist including `theory/` (commit `708ef49 fix - Allow music-rules to depend on music-theory in architecture\_rules test`), while `tests/theory/test\_architecture\_theory.c`'s allowlist stops at `theory/` itself and does \*\*not\*\* include `rules/` — i.e. the dependency is correctly one-directional.

&#x20;  - `Constraint` is a 5-variant tagged type (`HARD/SOFT/WEIGHTED/CONDITIONAL/CONTEXTUAL`), `constraint\_evaluate` is an explicit stub returning `CONSTRAINT\_EVAL\_NOT\_IMPLEMENTED`.

&#x20;  - `Preference.preference\_rank` has no PASS/FAIL path — cannot reject a candidate (`rules/preference.h:22-24`).

&#x20;  - `Framework` holds no back-reference into `music-interpretation` and no forward reference to any L7 sibling (`theory/framework.h`).

2\. \*\*Deferred, stated reasons\*\*: `RuleEvaluation` — flagged for "project-owner confirmation" since no distinct Registry row/shape was found (`docs/ARCHITECTURE.md:520-524`); `Procedure` deferred to L8 (wrong module); species-counterpoint and common-practice-harmony frameworks explicitly deferred as extension modules, citing TDD §244-245 ("No kernel changes should be permitted merely for convenience").

3\. \*\*Deviation / OPEN ISSUE §7.2 — verified directly, not resolved (see dedicated section below)\*\*.

4\. \*\*Silent decisions\*\*: none found.

\## L8 — Transform / Analysis / Generation (`src/transform`, `src/analysis`, `src/generation`)

&#x20;

Files: 12 transform files (`transformation.h` + 11 concrete subtypes), 10 analysis files (`analyzer.h`, `feature\_extraction.h` + 8 concrete analyzers), 9 generation files (`intent.h` through `generation\_record.h`, `candidate.h`, `candidate\_evaluator.h`).

&#x20;

1\. \*\*Implemented as spec'd\*\*:

&#x20;  - Sibling non-import verified directly: `tests/transform/test\_architecture\_transform.c`'s allowlist includes L0-L7 dirs plus `transform/` but \*\*not\*\* `analysis/` or `generation/` (`tests/transform/test\_architecture\_transform.c:47-53`); the same pattern holds for `tests/analysis/test\_architecture\_analysis.c` and `tests/generation/test\_architecture\_generation.c` (each allowlists only its own directory among the three L8 siblings). All three pass under `ctest`.

&#x20;  - `generation/candidate\_evaluator.h` does \*\*not\*\* `#include "analysis/analyzer.h"` despite `CandidateEvaluation` and `AnalysisResult` sharing an identical evaluations/violations-as-`Interpretation` shape — verified by direct `#include` grep (`src/generation/candidate\_evaluator.h:6-12` vs `src/analysis/analyzer.h:6-11`) — exactly the deliberate duplication `docs/ARCHITECTURE.md:627-633` describes.

&#x20;  - `Rule`/`Constraint`/`Preference`/`StatisticalModel` values reach L8 only as already-computed inputs (no `music-corpus` anywhere, consistent with the L7 open issue below).

&#x20;  - Referentially transparent Transformations: no mutator, no back-reference into an input object, confirmed by scanning `src/transform/\*.h` for mutator-style names (none found).

2\. \*\*Deferred, stated reasons\*\*: `ConstraintSolver`, `Search`/`Optimizer` (non-normative/reference concept); Hierarchical Generators (`ThemeGenerator`, `PhraseGenerator`, `BasicIdeaGenerator`, Registry Phase 13, wrong phase); `PracticeRequest`/`PracticeGenerator`, `DifficultyModel` (L9 territory).

&#x20;  - `ChordIdentification` and `KeyEstimation` deliberately do \*\*not\*\* construct a `Chord`/`Key` object (those don't exist at L6 yet) — they report a framework-relative `Interpretation` instead, mirroring `CadenceDetection` not constructing a `Cadence` and `MotifDetection` not persisting the `MotifOccurrence` it validates.

3\. \*\*Deviation\*\*: `framework\_version` is called out as a "KNOWN GAP, not fabricated" — TDD §163 requires it on `CandidateSet`/`GenerationRecord`, but `Framework` (L7) has no version field yet, so both types keep it as an optional/opaque nullable string (`docs/ARCHITECTURE.md:642-648`).

4\. \*\*Silent decisions\*\*: none found — this layer's notes section (`docs/ARCHITECTURE.md:576-666`) is the most detailed in the whole document and every shortcut is explained inline.

\---

&#x20;

\## Open Issue §7.1 — Interpretation-family ↔ Framework (VERIFIED, still unresolved as documented)

&#x20;

Registry lists `Interpretation`, `Chord`, `Key`, `ScaleDegree` as depending on `Framework` (L7) — a disallowed L6→L7 forward dependency if implemented literally.

&#x20;

Verification performed:

\- `grep -rn "#include" src/interpretation src/context src/semantics src/relationship` → \*\*zero\*\* hits on `theory/` or `rules/` headers (full listing captured during audit; only stdlib + `core/`, `identity/`, `time/`, `pitch/` includes appear).

\- `grep -rln "Framework" src/interpretation src/context src/semantics src/relationship` → two files mention the word, both in \*\*doc-comments only\*\*: `src/interpretation/interpretation.h:19,21` ("A literal import ... Interpretation has NO resolved framework field at all") and `src/semantics/sonority.h:38` ("\[Chord] is Chord (Phase 8, depends on Framework/L7, not built here)").

\- `grep -rln "FrameworkReference" src/` → \*\*no such type exists anywhere\*\* in `src/core` or `src/identity` (or anywhere else). So there is not even an opaque `FrameworkReference`-shaped ID at L0/L1 — the field was omitted entirely rather than stubbed.

\- `Chord`, `Key`, `ScaleDegree` are \*\*not implemented at all\*\* in the repo (no `chord.h`/`key.h`/`scale\_degree.h` under `src/interpretation` or `src/semantics`), so the disputed dependency has no implementation surface to violate for those three either.

\- `Framework` itself (`src/theory/framework.h`) holds no reference back into `music-interpretation` and no forward reference to any of its own L7 siblings.

\*\*Conclusion\*\*: the guardrail is being honored. No implementation currently establishes the Interpretation-family → Framework dependency. Status: still open/unresolved upstream, correctly not implemented downstream, and disclosed via a comment at the point where it would otherwise matter.

&#x20;

\## Open Issue §7.2 — Theory ↔ Corpus (VERIFIED, still unresolved as documented)

&#x20;

Registry lists `Tendency`, `StatisticalModel`, `StyleModel` (L7) as depending on `Corpus`/`Corpus Statistics` (L9, `music-corpus`) — a disallowed L7→L9 forward dependency.

&#x20;

Verification performed:

\- `grep -rn "#include" src/theory src/rules` → full listing captured; every include is either stdlib or `core/value.h`, `identity/entity\_id.h`, `theory/framework.h` (the one legal `rules`→`theory` case documented above). \*\*No `corpus/` or `music-corpus` include anywhere.\*\*

\- `find src -iname "\*corpus\*"` → \*\*no such directory/module exists in the repo at all\*\* (expected, since L9 is out of scope/not implemented).

\- `grep -rln -i "corpus" src/theory src/rules` → five files mention the word only in doc-comments/field names: `theory/tendency.h` (`training\_corpus` is documented as "a plain string label," not an object reference), `theory/statistical\_model.{h,c}`, `theory/style\_model.{h,c}` (same "plain label string" treatment).

\- Each of `Tendency`, `StatisticalModel`, `StyleModel` accepts corpus-derived numbers only as opaque, already-computed `Value` (`core/value.h`) fields or plain rational numerator/denominator fields — confirmed by reading `src/theory/tendency.h`, `statistical\_model.h`, `style\_model.h` in full.

\*\*Conclusion\*\*: the guardrail is being honored. No `music-corpus` module exists and nothing in `src/theory`/`src/rules` imports one; all corpus-shaped data enters as plain computed values. Status: still open/unresolved upstream, correctly not implemented downstream.

&#x20;

\---

&#x20;

\## Architecture-gate tests — verified

&#x20;

\*\*Existence\*\*: exactly 16 `test\_architecture\_\*.c` files found (`identity, core, time, pitch, events, containers, structure, context, semantics, relationship, interpretation, theory, rules, transform, analysis, generation`), matching the task's expected count.

&#x20;

\*\*Real enforcement, not stubs\*\*: Each file (`tests/<module>/test\_architecture\_<module>.c`) is a genuine directory-driven scanner (uses `dirent.h` to walk every `.h`/`.c` file under `src/<module>/`), extracts every local `#include "..."` line, and flags any include whose path prefix is not in that module's specific allowlist. This is a real allowlist-based import check, not a placeholder — confirmed by reading `tests/interpretation/test\_architecture\_interpretation.c` and `tests/transform/test\_architecture\_transform.c` in full.

&#x20;

\*\*L8 sibling restriction, specifically verified\*\*: `tests/transform/test\_architecture\_transform.c`'s allowlist (line \~47-53) includes L0-L7 directories plus `transform/` itself, but conspicuously omits `analysis/` and `generation/`. The same asymmetric-omission pattern holds in `tests/analysis/test\_architecture\_analysis.c` (omits `transform/`, `generation/`) and `tests/generation/test\_architecture\_generation.c` (omits `transform/`, `analysis/`). This is the stricter no-sideways-import rule the task asked about, and it is real (would actually fail the build if violated).

&#x20;

\*\*L7 asymmetric rule, specifically verified\*\*: `tests/rules/test\_architecture\_rules.c`'s allowlist includes `theory/` (rules may depend on theory); `tests/theory/test\_architecture\_theory.c`'s allowlist does \*\*not\*\* include `rules/` (theory may not depend on rules). This matches the one documented exception to L7's otherwise-mutual sibling restriction (`rules`→`theory` only, never the reverse).

&#x20;

\*\*Build \& test run — actually executed, not just inspected\*\*:

```

cmake -S . -B build -G "Unix Makefiles"   # succeeded

cmake --build build -j4                   # succeeded, exit 0, no warnings surfaced in tail output

ctest --test-dir build --output-on-failure

```

Result: \*\*100% tests passed, 0 failed, out of 95 total tests\*\* (including all 16 architecture-gate tests, run individually via `ctest -R architecture`: all 16 report `Passed`). The build system is CMake (`CMakeLists.txt` at root, `src/CMakeLists.txt`, `tests/CMakeLists.txt`), builds cleanly out of the box on this environment (GCC 13.3.0, C11) with no special configuration needed beyond `docs/BUILDING.md`'s documented two commands.

&#x20;

\---

&#x20;

\## Cross-cutting checks

&#x20;

\- \*\*Persistence never imported by L0-L8\*\*: `grep -rln -i "persistence\\|repository\\|sqlite\\|\\bSQL\\b" src/` → \*\*no hits at all\*\*. There is no persistence module in the repo yet (consistent with L9 not being implemented), and nothing under `src/` references one.

\- \*\*RelationshipReference resolution only at L6+\*\*: `RelationshipReference` (`src/core/relationship\_reference.{h,c}`) is defined at L1 but is not `#include`d or referenced anywhere else in `src/` yet (`grep -rln "RelationshipReference\\|relationship\_reference" src/` returns only its own definition files plus `src/CMakeLists.txt`). It has no resolution logic anywhere in the codebase currently — consistent with (and stronger than) the requirement that resolution not happen below L6, since it does not happen anywhere yet.

\- \*\*VoiceReference resolution timing\*\*: correctly documented as gaining "a real resolution target" only once `Voice` (L4) exists (`docs/ARCHITECTURE.md:267-270`); no resolution logic (a lookup function taking a `VoiceReference` and returning a `Voice`) exists anywhere in `src/containers` or elsewhere — the module notes explicitly call this out as intentionally out of scope (a registry/repository pattern is "an application/query-layer, L9, concern").

\- \*\*docs/ vs. code cross-reference\*\*: I found `docs/ARCHITECTURE.md`'s claims to be accurate everywhere I checked them against source (module dependency lists, deferred-type lists, the two open issues, the L8 sibling restriction, the `framework\_version` gap). I did not find a single instance where the doc claims something is implemented that isn't, or vice versa. The one gap in the docs is structural, not factual: `docs/ARCHITECTURE.md` does not mention the Domain Spec's "PROPOSED ONLY... MUST NOT be implemented until Registry synchronization" blocking language for `VoiceReference`/`RelationshipReference`/`ProvenanceReference` at all (see L1 section above) — it documents what these types do, but not the fact that a spec-level gate on building them existed.

\- \*\*TODO/FIXME/HACK/XXX scan\*\*: `grep -rn "TODO\\|FIXME\\|HACK\\|XXX\\b" src/ tests/` → \*\*zero hits\*\*. Every deferral in this codebase is expressed as prose ("Deferred", "not yet", "PROVISIONAL", "STATUS FLAG", "blocked", "PLACEHOLDER") rather than a code-level marker comment — an unusually disciplined convention, but worth noting since the task asked to grep for these specific tokens and they simply aren't used as a convention here.

\---

&#x20;

\## Notable silent decisions — roundup

&#x20;

Given how thoroughly disclosed this codebase is, the list of genuinely \*undocumented\* items is short. In order of how likely the user is to want to act on them:

&#x20;

1\. \*\*The Registry-synchronization blocker on `VoiceReference`/`RelationshipReference`/`ProvenanceReference` is never mentioned in-repo.\*\* (`src/core/voice\_reference.h`, `relationship\_reference.h`, `provenance\_reference.h`, all added in commit `bf00705`.) The types are implemented and thoroughly documented on their own terms, and the PR that introduced them predates none of the later work that depends on them (`ProvenanceReference` in particular is load-bearing for `NoteEvent`, `Relationship`, `Interpretation`, and more, throughout L3-L8). If the Domain Spec really does gate these on a Registry update that hasn't landed, the fact that three cross-cutting reference types — now deeply embedded across six layers — were built ahead of that gate, with zero acknowledgment of the gate anywhere in commit messages or comments, is the single highest-leverage finding in this audit. It is not "wrong" per se (nothing here contradicts the \*shape\* the Registry eventually needs), but it is a documented process gate that appears to have been silently bypassed rather than explicitly overridden with a rationale.

2\. \*\*`RelationshipReference` is defined but has zero consumers anywhere in the codebase.\*\* Every other L1 reference type (`VoiceReference` in `NoteEvent`, `ProvenanceReference` nearly everywhere) is actually used somewhere by the current HEAD; `RelationshipReference` is not referenced by any other file in `src/`. This isn't flagged as dead code or as "added for future use" anywhere — it's simply present and unused. Worth confirming this is intentional forward-provisioning rather than an oversight.

3\. \*\*`docs/ARCHITECTURE.md` has no top-level "Status" or "Known Gaps" index\*\* — all of the deferred/deviation information is scattered through per-module prose sections, which is thorough but means a reader has to read the whole 665-line file to reconstruct the full gap list this report assembles. Not a code issue, but worth flagging since the user is clearly trying to reconstruct exactly that list.

4\. \*\*`Container` (`src/containers/container.h`) has no `.c` file at all\*\* — it's a bare forward-declared opaque type with no fields, no constructor, nothing to compile. This is explicitly intentional per its own doc-comment ("no create/destroy or accessor functions -- there is no state here to construct or tear down"), so it is not silent, but it's an easy thing to mistake for an incomplete stub at a glance — worth being aware of when navigating the tree.

No other undocumented imports, mutator functions, float/double usages, missing fail-fast checks, or unexplained struct fields were found in the modules and files inspected during this pass.

&#x20;

