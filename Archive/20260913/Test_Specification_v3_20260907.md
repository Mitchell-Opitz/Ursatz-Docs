# TEST SPECIFICATION (Deliverable U) v3

Source: TDD v4 §193–207/235–237, Laws 1–25; Canonical IR Specification v3; Domain Specification v4; API Contract Specification v4; System Registry v3; Dependency & Layer Specification v3.
Supersedes: v2. Condensed — resolved blockers removed, test content unchanged.

## Status

Tests for `VoiceReference`, `RelationshipReference`, `ProvenanceReference` are runnable against real implementations now — Registry sync landed in Registry v3 (previously written-but-not-runnable). Remaining inherited open items: NoteEvent's category boundary (tests below still treat it under the provisional Event-category contract only), and TimeSpan's boundary vocabulary (tests cover only what's fixed).

## Purpose

Answers: **how will we know the implementation is correct?** Not what the objects are (Deliverable B) or how software touches them (Deliverable N/API Contract) — the tests that must exist and pass before an object/transformation/subsystem is considered correct. Absence of required coverage means the behavior is unverified.

Per TDD §201, required test categories:
```
Unit, Property-Based, Invariant, Golden, Serialization, Migration, Round-Trip,
Relationship, Provenance, Context, Theory, Transformation, Analysis Regression,
Generation Validation, Corpus Regression, Performance, Security, Fuzz
```
This document gives full treatment to the seven the current phase exercises — **Invariant, Property-Based, Serialization/Round-Trip, Provenance, Transformation, Relationship, Canonical Equality** (equality folded in per Registry) — plus Architecture tests (required by TDD §193, not its own §201 row). The rest are stubbed in §Deferred Categories with their owning future deliverable.

---

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

---

## Invariant tests

**Rule (API Contract C3):** every constructor validates invariants synchronously, raises `InvariantViolation` on violation — no "construct now, validate later." Every test below is construction-time.

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
| ProvenanceReference | `state` always explicit, never defaults to absent on omission; `state=present` requires a syntactically valid `provenance_id` (non-dangling resolution tested by Deliverable K's resolution layer, not this constructor); `state=absent` requires no `provenance_id` |

**Fail-fast test:** for every object type, feed a payload that fails validation partway through a multi-field constructor → assert no object instance is observable anywhere (not returned, not partially stored, not cached) — only `InvariantViolation` propagates. This directly tests C3, distinct from just checking the exception fires.

---

## Property-based tests

Foundational value objects use property tests over example tests wherever Domain Spec text uses "any," "never," or "always" language — claims a handful of examples can't falsify.

```
Duration:
  ∀ a,b,k (k>0 integer): Duration(a,b) == Duration(k·a, k·b)
  ∀ valid constructor path: result.value >= 0
  every successful construction stores/serializes the unique reduced rational
    (reduction is a construction postcondition, not just an equality-operator behavior)

MusicalTime:
  ∀ t: Duration, dt: Duration: exact(t + dt) == exact-rational-sum(t, dt)
  no float/double appears at any point in the computation
  exact equality/arithmetic tested WITHOUT assuming reduced-storage semantics
    (Duration's reduction postcondition doesn't automatically apply here)

PitchSpelling:  ∀ s: deserialize(serialize(s)) == s, no collapse to an enharmonic equivalent
EntityID:       ∀ id: value stable under serialization round-trip

Transposition (stubbed until Deliverable M lands):
  augmentation by factor 1 preserves value
  double inversion under the same axis restores original pitch
  retrograde applied twice restores original sequence
```

General pattern for any new Value object: identify the algebraic/semantic law its Domain Spec section claims → generate random valid inputs (not hand-picked) and assert the law holds → assert the law's negation does NOT spuriously pass (mutation-test the property itself).

---

## Serialization round-trip tests

Per API Contract: every `.serialize()`/`.deserialize()` is an object-level, abstract round-trip contract (field presence, explicitness, round-trip stability) — not a wire-format spec (Deliverable O).

General pattern:
```
∀ validly-constructed o: deserialize(serialize(o)) is canonically equal to o
serialize(o) contains every required field, no silent omission (esp. provenance state)
deserialize(malformed/incomplete payload) raises InvariantViolation, never silently
  defaults a required field (e.g. ProvenanceReference.state MUST NOT default to absent)
round-trip preserves the present/absent distinction on ProvenanceReference without
  collapsing to a single "no provenance" case
```

**Composite graph round trip (TDD §235, future/expanded scope):** currently testable — events (NoteEvent), voice (Voice), provenance (ProvenanceReference). Provisionally specified but not implementation-complete — sonority, context, interpretations, relationships. Verify after deserialize, as each component lands: identity preserved, version preserved, canonical equality preserved, relationships preserved, interpretations preserved (non-authoritative status unchanged), context preserved, provenance preserved.

**Non-goals:** byte layout/wire encoding → Deliverable O. Storage row shape → Deliverable P. Cross-version schema migration → Migration category (Deferred, below).

---

## Provenance tests

Grounded in Domain Spec's ProvenanceReference three-state model, TDD §198, Law 10 (provenance preserved), Law 20 (transformations derive, never mutate).

**Invariant-level:**
```
ancestor references in a Derivation exist (no dangling ancestor)
activity references exist; agent references exist
mandatory derivation edges exist where the schema requires them
provenance graph is acyclic where DAG semantics are declared
a provenance graph containing an unresolved state=present reference fails
  Provenance graph validation/resolution (Deliverable K) — a graph/resolution-layer
  test, not a constructor-level test
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
N1–N3 are structurally unchanged after producing N4–N6 — no observable canonical
  field modified (domain-level structural check, independent of serialization)
```

---

## Transformation tests

Grounded in TDD §146/§200, Law 20.

**Required validations per transformation:** input compatibility, parameter validity, domain invariant preservation on output, instrument constraints where applicable, output validity (passes the same Invariant tests as any hand-constructed instance), provenance completeness, event mapping correctness.

**Non-mutation test (the load-bearing one):**
```
∀ transformation T, input x: T(x) does not alter any observable field of x
  (compare by full structural equality, not just EntityID)
a transformation producing a new entity MUST assign a new EntityID; a revision
  operation MUST preserve the source EntityID and create a new Version instead —
  the test determines which case applies from the operation's declared semantics
```

**Golden tests (TDD §203):** critical/named transformations get version-controlled fixtures, provenance included in expected output:
```
Input: C E G. Operation: transpose +P5.
Expected: output pitches G4/B4/D5; distinct new EntityIDs; provenance state=present,
  resolving to the expected derivation/activity (once Deliverable K/M exist);
  correct source-to-output derivation mapping (N1→N4, N2→N5, N3→N6);
  input objects structurally unchanged
```
Golden fixtures are regression tests, not correctness proofs — they catch drift; the sections above establish correctness.

**Reproducibility (Law 21, stubbed — no stochastic transform exists yet):** stochastic transformation accepts an explicit seed; same seed + same input → byte-identical output across runs.

---

## Relationship validity tests

Grounded in TDD §79/§197. `Relationship` (L6, full edge) and `RelationshipReference` (L1, opaque pointer) get different treatment.

**Structural invariants:** source/target endpoint exists (non-dangling); predicate exists and permits the source's/target's actual type; required context/provenance exists where the predicate's contract demands one.

**Temporal/versioning metadata:** a Relationship may carry valid_from/valid_until/context/framework/version; a Relationship whose interpretation changes over time is a NEW version, not a mutation (Laws 6/8); two Relationships with the same source/target/predicate but different context/framework are both retrievable, neither silently overwrites the other.

**Reference opacity (`RelationshipReference`, API Contract C4):**
```
no constructor/query on an L0–L4 object resolves a RelationshipReference into its
  target — resolution only exposed at L6+
RelationshipReference carries none of the structural-invariant fields (predicate/
  scope/context/evidence) — those live only on the resolved Relationship
supplying a resolved Voice/Relationship/Provenance object where an opaque reference
  is required MUST fail type/contract validation, never silently auto-convert
```

---

## Canonical equality tests

The Registry names five equality concepts (`IdentityEquality`, `ValueEquality`, `StructuralEquality`, `SemanticEquality`, `CanonicalEquality`). Law 23 (canonicalization is not equality) is why these are tested separately rather than collapsed into one `==`. Each must be independently testable; implications between them are asserted only where a specific Domain Spec section already establishes them — elsewhere marked provisional, not invented here.

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
  normalization policy V compare canonical-equal under V; the same two objects MAY
  compare canonical-UNEQUAL under V' (policy version is test input, not incidental);
  canonical-equal does NOT imply identity-equal
```

Cross-notion regression: for a fixed pair of objects, assert all five checks independently; assert only implications already established by an authoritative source — the full algebra between the five remains open pending a dedicated equality spec.

---

## Architecture (boundary) tests

Per TDD §193 — automated, run in CI against the actual module graph, not review-based.

```
music-core (L1) does not import music-events (L3) or higher
pitch module does not import harmony module
time module does not import notation module
domain modules do not import persistence modules or SQL/storage-specific code
theory/framework modules cannot mutate canonical events (static check)
plugins cannot bypass capability boundaries
no L1–L4 object's module imports the module of anything it only holds by opaque
  reference (VoiceReference/RelationshipReference/ProvenanceReference) — the
  reference-only dependency rule made into a CI check
```

This enforces the Dependency & Layer Specification directly and is required regardless of how the Registry's Canonical IR row is eventually annotated — independent of that open issue.

---

## Deferred categories

Named in TDD §201, no Phase-1 object to bind to yet:

| Category | Deferred to | Why not testable yet |
|---|---|---|
| Context | Deliverable H | Context composition not yet specified |
| Theory | Deliverable L | No Rule/Constraint objects exist in Phase-1 |
| Analysis Regression | analysis phase | No analyzer interface has Phase-1 domain material |
| Generation Validation | Generation phase | No Intent/Constraint objects specified |
| Corpus Regression | Post-kernel | Requires a retained representative corpus |
| Performance | Ongoing, cross-cutting | No performance budget defined yet |
| Security/Fuzz | Deliverable T / I-O phase | Requires Deliverable O's wire format to exist first |
| Migration | Deliverable P / first schema change | No second schema version exists yet |

---

## Coverage

All seven fully-treated categories (Invariant, Property-Based, Serialization/Round-Trip, Provenance, Transformation, Relationship, Canonical Equality) plus Architecture tests are specified for the Phase-1 slice. Blocked on: Deliverable G (Relationship object), Deliverable K (Provenance record shape), Deliverable M (Transformation Architecture), Deliverable O (wire format — object-level fuzzing doesn't require waiting for this, wire-format-specific Security/Fuzz does).
