# Test Specification (Deliverable U)

**Last verified against repo state:** 2026-09-14 (self-audit pass, no repo changes since 2026-09-13). This is a Phase-1 kernel spec describing
required test categories and their treatment; unchanged in substance since — the Framework
Stage session extended coverage (see `Status.md`'s note on architecture-guard tests) without
changing this document's contract.

Source: Technical Design Document, Laws 1–25, Canonical IR Specification, Domain
Specification, API Contract Specification, System Registry, Dependency & Layer
Specification.

## Purpose

Answers: **how will we know the implementation is correct?** Not what the objects are
(Domain Spec) or how software touches them (API Contract/Canonical IR) — the tests that must
exist and pass before an object/transformation/subsystem is considered correct. Absence of
required coverage means the behavior is unverified.

Required test categories:
```
Unit, Property-Based, Invariant, Golden, Serialization, Migration, Round-Trip,
Relationship, Provenance, Context, Theory, Transformation, Analysis Regression,
Generation Validation, Corpus Regression, Performance, Security, Fuzz
```
This document gives full treatment to the categories the current phase exercises —
**Invariant, Property-Based, Serialization/Round-Trip, Provenance, Transformation,
Relationship, Canonical Equality** — plus Architecture tests (required, not its own category
row). The rest are stubbed in Deferred Categories with their owning future deliverable.

## Test category taxonomy

| Category | Answers | Applies to |
|---|---|---|
| Invariant | Can this object ever exist in an illegal state? | Every object with Domain Spec invariants |
| Property-Based | Does a general rule hold across the input space, not just examples? | Value objects, arithmetic, symmetric ops |
| Serialization/Round-Trip | Does `deserialize(serialize(x)) == x` under canonical equality? | All 10 Phase-1 objects + 3 reference types |
| Provenance | Is lineage present, correct, non-dangling wherever required? | Derived/generated objects; `ProvenanceReference` |
| Transformation | Does an op preserve invariants and never mutate its input? | Any op producing a new object from existing ones |
| Relationship | Are edges valid, with required temporal/context metadata? | `Relationship`, `RelationshipReference` |
| Canonical Equality | Do two representations of "the same fact" compare equal under the right notion? | Cross-cutting |
| Architecture | Does the code obey layer/import boundaries? | Module graph |
| Golden | Does a critical transformation reproduce a version-controlled expected output? | Named transformations |
| Unit | Does one function do the one thing its contract says? | All (baseline) |

## Invariant tests

**Rule (API Contract C3):** every constructor validates invariants synchronously, raises
`InvariantViolation` on violation — no "construct now, validate later." Every test below is
construction-time.

General pattern, applied per object:
```
construct with each individually-violated invariant → raises InvariantViolation
construct with all invariants satisfied → succeeds
no code path (alternate constructor, deserialize, copy) bypasses validation
```

Per-object invariants under test:

| Object | Invariant |
|---|---|
| EntityID | non-empty; no positional leakage; stable across all revisions |
| Duration | `>= 0`; denominator `> 0`; never float/double |
| MusicalTime | exact rational; arithmetic preserves exactness |
| TimeSpan | `end >= start`; `boundary` required, no implicit default |
| PitchIdentity | construction doesn't require tuning params; `token` not EntityID-typed |
| PitchSpelling | two enharmonically-equivalent spellings are NOT value-equal |
| NoteEvent | `duration >= 0`; provenance state explicitly present/absent, never omitted; `optional_voice`, if present, is a VoiceReference and nothing else |
| Voice | does not carry an implicit Instrument/Staff/Part field in required schema |
| VoiceReference | carries no membership/containment fields (opaque pointer only) |
| RelationshipReference | carries no predicate/scope/context/evidence fields |
| ProvenanceReference | `state` always explicit, never defaults to absent on omission; `state=present` requires a syntactically valid `provenance_id`; `state=absent` requires no `provenance_id` |

**Fail-fast test:** for every object type, feed a payload that fails validation partway
through a multi-field constructor → assert no object instance is observable anywhere (not
returned, not partially stored, not cached) — only `InvariantViolation` propagates.

## Property-based tests

Foundational value objects use property tests over example tests wherever Domain Spec text
uses "any," "never," or "always" language.

```
Duration:
  ∀ a,b,k (k>0 integer): Duration(a,b) == Duration(k·a, k·b)
  ∀ valid constructor path: result.value >= 0
  every successful construction stores/serializes the unique reduced rational

MusicalTime:
  ∀ t: Duration, dt: Duration: exact(t + dt) == exact-rational-sum(t, dt)
  no float/double appears at any point in the computation
  exact equality/arithmetic tested WITHOUT assuming reduced-storage semantics

PitchSpelling:  ∀ s: deserialize(serialize(s)) == s, no collapse to an enharmonic equivalent
EntityID:       ∀ id: value stable under serialization round-trip

Transformation (now implemented — Phase 7 done):
  augmentation by factor 1 preserves value
  double inversion under the same axis restores original pitch
  retrograde applied twice restores original sequence
```

General pattern for any new Value object: identify the algebraic/semantic law its Domain
Spec section claims → generate random valid inputs (not hand-picked) and assert the law
holds → assert the law's negation does NOT spuriously pass.

## Serialization round-trip tests

```
∀ validly-constructed o: deserialize(serialize(o)) is canonically equal to o
serialize(o) contains every required field, no silent omission (esp. provenance state)
deserialize(malformed/incomplete payload) raises InvariantViolation, never silently
  defaults a required field (e.g. ProvenanceReference.state MUST NOT default to absent)
round-trip preserves the present/absent distinction on ProvenanceReference without
  collapsing to a single "no provenance" case
```

**Composite graph round trip:** currently testable — events (NoteEvent), voice (Voice),
provenance (ProvenanceReference), and now PitchSpelling (persisted end-to-end since Ursatz
PR #38). Not yet complete for sonority, context, interpretations, relationships.

**Non-goals:** byte layout/wire encoding, storage row shape, cross-version schema migration.

## Provenance tests

**Invariant-level:**
```
ancestor references in a Derivation exist (no dangling ancestor)
activity references exist; agent references exist
mandatory derivation edges exist where the schema requires them
provenance graph is acyclic where DAG semantics are declared
```

**Lifecycle-level:**
```
every object produced by a Transformation/Activity carries state=present with a
  resolving provenance_id
state=absent is explicitly constructible via .absent(absence_reason) wherever
  Domain Spec permits absence; MUST NOT be used for Activity output
a derived object receives a NEW EntityID, never a revision of its source's EntityID
```

**Worked-example regression** (C4–E4–G4 triad + P5 transposition, fixed fixture):
```
N1, N2, N3 (hand-authored) carry state=absent with a non-empty absence_reason
N4, N5, N6 (transposed) carry state=present, provenance_id resolving to a
  Derivation naming Transposition(+P5), derived_from pointing at N1/N2/N3 respectively
N1–N3 are structurally unchanged after producing N4–N6
```

## Transformation tests

**Required validations per transformation:** input compatibility, parameter validity, domain
invariant preservation on output, instrument constraints where applicable, output validity,
provenance completeness, event mapping correctness.

**Non-mutation test (the load-bearing one):**
```
∀ transformation T, input x: T(x) does not alter any observable field of x
a transformation producing a new entity MUST assign a new EntityID; a revision
  operation MUST preserve the source EntityID and create a new Version instead
```

**Golden tests:** critical/named transformations get version-controlled fixtures, provenance
included in expected output. Golden fixtures are regression tests, not correctness proofs.

**Reproducibility (Law 21):** stochastic transformation accepts an explicit seed; same seed +
same input → byte-identical output across runs. Still no stochastic transform exists in the
codebase, so this remains stubbed.

## Relationship validity tests

`Relationship` (L6, full edge) and `RelationshipReference` (L1, opaque pointer) get different
treatment. **Now exercised by real production code** (Period/Sentence/Form/Thematic Return
Detection, each shipped with its own regression coverage) — not just a specified contract.

**Structural invariants:** source/target endpoint exists (non-dangling); predicate exists and
permits the source's/target's actual type; required context/provenance exists where the
predicate's contract demands one.

**Temporal/versioning metadata:** a Relationship may carry valid_from/valid_until/context/
framework/version; a Relationship whose interpretation changes over time is a NEW version,
not a mutation; two Relationships with the same source/target/predicate but different
context/framework are both retrievable, neither silently overwrites the other.

**Reference opacity (`RelationshipReference`, API Contract C4):**
```
no constructor/query on an L0–L4 object resolves a RelationshipReference into its
  target — resolution only exposed at L6+
RelationshipReference carries none of the structural-invariant fields
supplying a resolved Voice/Relationship/Provenance object where an opaque reference
  is required MUST fail type/contract validation, never silently auto-convert
```

## Canonical equality tests

Five equality concepts (`IdentityEquality`, `ValueEquality`, `StructuralEquality`,
`SemanticEquality`, `CanonicalEquality`). Law 23 (canonicalization is not equality) is why
these are tested separately rather than collapsed into one `==`.

```
IdentityEquality: two refs to the same EntityID+Version compare equal regardless of
  in-memory instance; two objects with same field values but different EntityIDs
  are NOT identity-equal
ValueEquality: Duration(3,4) == Duration(6,8); two enharmonically-equivalent
  PitchSpellings are NOT value-equal (negative case, explicit non-collapse)
StructuralEquality: two graphs with isomorphic node/edge shape but different
  EntityIDs compare structurally equal under an explicit rule set, NOT identity-equal
SemanticEquality: two representations under different Context/Tuning denoting the
  same musical fact compare semantically equal, without asserting same object
CanonicalEquality: two objects canonicalizing to the same normal form under
  normalization policy V compare canonical-equal under V; canonical-equal does NOT
  imply identity-equal
```

## Architecture (boundary) tests

Automated, run in CI against the actual module graph, not review-based.

```
music-core (L1) does not import music-events (L3) or higher
pitch module does not import harmony module
time module does not import notation module
domain modules do not import persistence modules or SQL/storage-specific code
theory/framework modules cannot mutate canonical events (static check)
plugins cannot bypass capability boundaries
no L1–L4 object's module imports the module of anything it only holds by opaque
  reference — the reference-only dependency rule made into a CI check
```

**Current coverage (verified 2026-09-13):** every analyzer added in the Framework Stage
session shipped with its own architecture guard test; full suite is well over 150 tests and
growing with each PR through #46.

## Deferred categories

| Category | Deferred to | Why not testable yet |
|---|---|---|
| Context | Future Context spec | Context composition not yet specified |
| Theory | Future Theory/Rule/Constraint spec | Constraint/Preference remain deliberate stubs |
| Analysis Regression | — | **Partially addressed**: analyzer-level unit/regression tests exist per analyzer, but the full-corpus verification pass (`framework-v1-reference-set`, all 5 pieces, output checked not just registration) is still the open item — see `Overview/Master_Roadmap.md` |
| Generation Validation | Generation phase | `Intent`/`ConstraintExtraction`/`Candidate`/etc. exist as real base types (`src/generation/`), but no `Generator`/`IntentCompiler` orchestration exists to produce output worth regression-testing — see `Registry/Registry_Theory_Analysis.md` |
| Corpus Regression | — | `CorpusRegression` test exists but verifies registration only, not analysis output |
| Performance | Ongoing, cross-cutting | No performance budget defined yet |
| Security/Fuzz | Future I/O deliverable | Requires a formal wire format to exist first |
| Migration | Future Persistence deliverable | No second schema version exists yet |

## Coverage

All seven fully-treated categories (Invariant, Property-Based, Serialization/Round-Trip,
Provenance, Transformation, Relationship, Canonical Equality) plus Architecture tests are
specified for the Phase-1 slice and exercised in practice by the full test suite (150+ tests
as of PR #46). Still blocked on: a formal Relationship spec, a formal Provenance record-shape
spec, a formal Transformation Architecture spec, and a formal wire-format spec — all
implemented in code ahead of their owning written spec, a disclosed gap (see `Status.md`).
