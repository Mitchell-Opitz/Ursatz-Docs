\# Ursatz Cleanliness Audit (size / verbosity / maintainability only)

&#x20;

Scope: read-only audit, no correctness/architecture judgment. Numbers below come from

direct greps/counts over the working tree on 2026-09-04.

&#x20;

Repo shape: 81 headers + 78 `.c` files under `src/` (8,283 lines of implementation),

95 test `.c` files under `tests/` (8,628 lines), `docs/ARCHITECTURE.md` (665 lines),

3 `CMakeLists.txt` files.

&#x20;

\---

&#x20;

\## 1. Comment density in headers

&#x20;

\*\*Raw numbers\*\* (all 81 `.h` files, 5,309 total lines):

&#x20;

| Category | Lines | % of total |

|---|---|---|

| Code (declarations) | 1,957 | 37% |

| Comments | 2,651 | 50% |

| Blank | 701 | 13% |

&#x20;

Of the 2,651 comment lines, only \*\*61 are standalone one-line comments\*\* (`/\* ... \*/`

on a single line); the remaining \~2,590 lines belong to \*\*multi-paragraph block

comments\*\* above types/constructors. There are \*\*zero\*\* license/copyright/file-banner

comments anywhere in `src/` — no boilerplate category exists in this codebase at all.

&#x20;

\*\*Qualitative finding, sampled across every requested module\*\* (identity, core, time,

pitch, events, containers, structure, semantics, context, relationship,

interpretation, theory, rules, analysis, generation, transform):

&#x20;

The documentation style is unusually disciplined and consistent. The large majority of

comment blocks are genuinely load-bearing: they cite a spec section (`TDD §NN`,

`Registry #N`, `Dependency \& Layer Spec v2 §N`), state an ownership rule

("takes ownership of X on success"), or flag a known gap/open issue explicitly

("KNOWN GAP", "OPEN ISSUE", "contested and unresolved upstream"). Very little is filler

that just restates a function signature — searches for the classic anti-pattern

(`/\* Returns the X \*/` above a getter, `/\* Sets X \*/` above a setter) turned up \*\*zero

matches\*\* anywhere in `src/\*.h`. Most accessors are simply left uncommented because the

name is self-explanatory, which is the correct choice, not a gap.

&#x20;

Estimated split of the 2,651 comment lines:

\- \*\*\~85-90% load-bearing\*\* — invariants, ownership rules, spec citations, deferred/open

&#x20; items.

\- \*\*\~8-12% restating-the-obvious or purely formulaic\*\* — mostly a small number of

&#x20; one-line comments that are repeated \*verbatim\* across sibling files describing an

&#x20; identical field shape (see examples 5-6 below). Individually each is fine; as a set

&#x20; across near-identical files it reads as boilerplate.

\- \*\*\~0% license/banner boilerplate\*\* — none exists.

\### Concrete examples

&#x20;

\*\*Must stay — explains a real invariant / deferred item (load-bearing):\*\*

&#x20;

1\. `src/generation/candidate.h:118-125` — flags a genuine cross-module gap rather than

&#x20;  silently fabricating a field:

&#x20;  ```

&#x20;  \*   - framework\_version is a KNOWN GAP, flagged rather than silently

&#x20;  \*     fabricated: TDD Sec163 requires it, but Framework (theory/

&#x20;  \*     framework.h) carries no version field of its own yet -- the same

&#x20;  \*     unresolved-dependency situation Interpretation flags for its own

&#x20;  \*     omitted framework field. Kept here as an optional, opaque

&#x20;  \*     descriptor string (NULL when not available) until Framework

&#x20;  \*     gains real versioning; not required, so this type does not block

&#x20;  \*     on that gap.

&#x20;  ```

&#x20;  Deleting this would silently lose the reason a field is a bare string instead of a

&#x20;  real reference — a future maintainer would have no way to know this is intentional

&#x20;  and temporary.

2\. `src/theory/tendency.h:9-19` — states a hard behavioral constraint that isn't visible

&#x20;  from the type declaration at all:

&#x20;  ```

&#x20;  \* A Tendency's frequency is an exact rational fraction (numerator/

&#x20;  \* denominator, Law 19, never float/double) describing how often the

&#x20;  \* observed behavior occurs -- and nothing more. It MUST NEVER

&#x20;  \* automatically imply must/must-not/valid/invalid: this header defines

&#x20;  \* no PASS/FAIL/OK-as-validity vocabulary anywhere, deliberately...

&#x20;  ```

&#x20;  This is the difference between "what the struct holds" and "what it must never be

&#x20;  used for" — not recoverable from the signature.

3\. `src/rules/rule.h:15-19` — explains an ownership/lifetime rule that would otherwise

&#x20;  be a use-after-free trap:

&#x20;  ```

&#x20;  \* framework is a reference only, not owned -- the same reference-only

&#x20;  \* pattern Context uses for its parent (context/context.h): rule\_destroy

&#x20;  \* never destroys a Rule's framework, since the same Framework is

&#x20;  \* expected to be referenced by many Rules/Constraints/Preferences.

&#x20;  ```

&#x20;

\*\*Could be trimmed — restates the obvious or is formulaic boilerplate:\*\*

&#x20;

4\. `src/structure/motif\_occurrence.h:52`:

&#x20;  ```c

&#x20;  /\* The id of the Motif this is an occurrence of. \*/

&#x20;  const EntityID \*motif\_occurrence\_motif\_id(const MotifOccurrence \*motif\_occurrence);

&#x20;  ```

&#x20;  The function name plus the struct-level doc comment 20 lines above it already say

&#x20;  this; the one-liner adds essentially nothing beyond what `\_motif\_id` already

&#x20;  communicates.

5\. Six near-identical one-liners repeated verbatim across `src/analysis/\*.h`

&#x20;  (`cadence\_detection.h:76`, `chord\_identification.h:82`, `interval\_analysis.h:62`,

&#x20;  `key\_estimation.h:75`, `motif\_detection.h:91`, `phrase\_segmentation.h:79`,

&#x20;  `scale\_analysis.h:68`):

&#x20;  ```c

&#x20;  /\* The Analyzer value pluggable wherever an Analyzer is expected. \*/

&#x20;  ```

&#x20;  Same wording, same position, seven times. It's not wrong, but it's the kind of

&#x20;  comment that a single "Analyzer conformance" note in `analysis/analyzer.h` could

&#x20;  cover once, with each sibling header simply not repeating it.

6\. Ten near-identical one-liners across every `src/transform/\*.h` file (`augmentation.h:71`,

&#x20;  `contraction.h:63`, `diminution.h:61`, `displacement.h:61`, `expansion.h:63`,

&#x20;  `fragmentation.h:60`, `inversion.h:73`, `ornamentation.h:84`, `retrograde.h:71`,

&#x20;  `transposition.h:76`):

&#x20;  ```c

&#x20;  /\* The owned base Transformation -- input/output type, dimensions, mapping behavior, determinism, provenance. \*/

&#x20;  ```

&#x20;  Same pattern as #5 — correct and harmless, but purely formulaic repetition rather

&#x20;  than information specific to the file it's in.

\---

&#x20;

\## 2. `MUSIC\_` prefix convention

&#x20;

Grepped every occurrence of `MUSIC\_` across `src/\*\*/\*.h` and `src/\*\*/\*.c`.

&#x20;

\*\*Finding: `MUSIC\_` is used exactly once per header, twice counting the matching

`#endif`-adjacent `#define`/`#ifndef` pair, and \*nowhere else in the codebase\*.\*\*

&#x20;

\- Occurrences: 160 total, all of the form `#ifndef MUSIC\_<MODULE>\_<NAME>\_H` /

&#x20; `#define MUSIC\_<MODULE>\_<NAME>\_H` — exactly 2 per header × 80 headers = 160.

\- \*\*Zero\*\* struct typedefs, enum tags, macros, or function names anywhere carry a

&#x20; `MUSIC\_` prefix. Public function names use plain module-scoped prefixes instead

&#x20; (`entity\_id\_create`, `candidate\_set\_destroy`, `motif\_occurrence\_motif\_id`, etc.),

&#x20; which is the correct, idiomatic approach for a C library with no namespaces.

\- \*\*Zero\*\* `static` (internal-linkage) symbols use `MUSIC\_` (or any project-wide

&#x20; prefix) — verified by scanning all 62 `static` functions across `src/\*.c`; none of

&#x20; them are prefixed beyond a normal short local name.

\- \*\*Consistency gap found:\*\* `src/ursatz.h`, the single umbrella header at the

&#x20; package root, is the \*one\* header of 81 that breaks the convention — its guard is

&#x20; `#ifndef URSATZ\_H` / `#define URSATZ\_H`, not `MUSIC\_URSATZ\_H`. Everything else

&#x20; (80/81 headers) is uniform.

\*\*Conclusion:\*\* the premise that `MUSIC\_` might be over-applied to internal/static

symbols does not hold — it isn't used there at all. It is applied narrowly and

correctly, exactly where it does real collision-prevention work (header guards), and

is essentially free of verbosity concerns. The only actionable item is the single

inconsistent guard in `ursatz.h`.

&#x20;

Counts: \*\*0 static functions incorrectly prefixed, 0 public symbols missing a

namespacing prefix pattern (all public functions/types use their module prefix

correctly), 1 header (1.2%) with an inconsistent guard name.\*\*

&#x20;

\---

&#x20;

\## 3. General repo size / structure

&#x20;

\*\*Repeated multi-line patterns in `.c` files:\*\*

&#x20;

\- `if (x == NULL) { ... }` null-guard blocks: \*\*400\*\* occurrences across `src/\*.c`.

&#x20; This is idiomatic defensive C (each function validates its own required pointer

&#x20; arguments, since there are no exceptions), and each guard returns a

&#x20; \*type-specific\* error enum value (`ENTITY\_ID\_ERR\_ALLOC`,

&#x20; `CANDIDATE\_ERR\_NULL\_MATERIAL\_ID`, etc.), so a generic macro would either lose the

&#x20; specific error code or need a parameter per call site, netting little real

&#x20; simplification.

\- `malloc(sizeof(...))` allocation sites: \*\*98\*\*, one per constructor, each followed

&#x20; by its own allocation-failure check and field population — again a per-type

&#x20; create/destroy pair, not accidental duplication so much as the standard shape of

&#x20; a C ADT library repeated deliberately 78 times (once per type file).

\- `\_destroy()` functions with the "if NULL return; free each owned member; free(ptr)"

&#x20; shape: \*\*68\*\* functions, same story — structurally identical but semantically

&#x20; distinct per type (each frees a different set of owned members).

None of these three patterns is a case of \*accidental\* copy-paste drift; they're the

natural, repeated shape of a hand-rolled ADT-per-file C library. A shared "constructor

helper" macro is technically possible but would trade a small amount of boilerplate

for a meaningfully harder-to-read/debug macro-expanded control flow — see

Recommendations.

&#x20;

\*\*File naming / directory structure:\*\* no inconsistencies found. Every `.c` file has

a matching `.h` except three pure-marker/category headers with no functions to

implement (`containers/container.h`, `generation/generator.h`,

`generation/intent\_compiler.h` — these define categories/interfaces only, correctly

have no `.c`). `tests/` mirrors `src/`'s module directories 1:1 (16 module

subdirectories). No stray naming conventions, casing mismatches, or misplaced files

were found in either tree.

&#x20;

\*\*`docs/ARCHITECTURE.md`:\*\* 665 lines, 11 top-level `##` headings: `Layers`,

`Modules`, then one section per L0-L5 module (`music-core`, `music-time`,

`music-pitch`, `music-events`, `music-containers`, `music-structure`), then three

\*lumped\* sections covering multiple modules each for L6 (`context` + `semantics` +

`relationship` + `interpretation`, 112 lines), L7 (`theory` + `rules`, 98 lines), and

L8 (`transform` + `analysis` + `generation`, 122 lines). There is no table of

contents and no status/gaps index (already flagged separately per your prior audit —

not re-derived here). At 665 lines this document is not yet "monolithic" in an

extreme sense, but the per-layer section granularity is uneven (single-module

sections at L0-L5 vs. 3-4-modules-per-section at L6-L8), and with no navigation aid a

reader has to scroll/search to find anything. Splitting the whole file per-layer

would be more restructuring than the current size actually justifies; a cheaper fix

—adding a top-of-file table of contents plus breaking the three lumped L6/L7/L8

sections into one heading per module (matching the L0-L5 pattern) — would get most of

the navigability benefit for a fraction of the effort of a full split.

&#x20;

\*\*Other size observations:\*\* no file in `src/` exceeds 359 lines

(`generation/candidate.c`); no file mixes an unusual number of concerns. No

license/file-header banners exist anywhere to trim. Nothing else stood out as

non-behavioral "bigness" beyond what's covered above.

&#x20;

\---

&#x20;

\## 4. CMake / build / test structure

&#x20;

\- Top-level `CMakeLists.txt`: 9 lines, minimal, nothing to flag.

\- `src/CMakeLists.txt`: 82 lines, a single `add\_library(ursatz ...)` call listing all

&#x20; 78 `.c` files explicitly. This is a deliberate, reasonable choice (explicit file

&#x20; lists are generally preferred over `file(GLOB ...)` in CMake for reliable

&#x20; reconfiguration) — not a verbosity problem.

\- `tests/CMakeLists.txt` + 16 per-module `tests/<module>/CMakeLists.txt` files:

&#x20; \*\*344 lines total\*\*, and this is the one genuinely repetitive spot. Every one of

&#x20; the 95 unit tests is wired up with the identical 3-line block:

&#x20; ```cmake

&#x20; add\_executable(test\_X test\_X.c)

&#x20; target\_link\_libraries(test\_X PRIVATE ursatz)

&#x20; add\_test(NAME X COMMAND test\_X)

&#x20; ```

&#x20; repeated verbatim \~95 times, plus a 16th near-identical 2-line variant per module

&#x20; for the `test\_architecture\_<module>` case. A small CMake function

&#x20; (e.g. `add\_unit\_test(name)`) or a `foreach(name IN ITEMS ...)` loop per module

&#x20; could collapse this to roughly a fifth of the current line count with no loss of

&#x20; clarity — this is the single clearest "same pattern copy-pasted N times" finding

&#x20; in the whole audit.

\---

&#x20;

\## Recommendations (prioritized, my judgment)

&#x20;

1\. \*\*Highest win / lowest risk — collapse the `tests/\*/CMakeLists.txt` boilerplate.\*\*

&#x20;  This is the most mechanical, most repetitive, least risky item in the whole audit:

&#x20;  pure build-system text, no behavior, no header/comment content, 16 files that all

&#x20;  follow one 3-line pattern \~95 times over. A `function(add\_unit\_test name)` wrapper

&#x20;  (or a per-module `foreach`) would cut \~344 lines to well under 100 with zero

&#x20;  ambiguity about what changed. I'd do this first.

2\. \*\*Second — fix the one `MUSIC\_` guard inconsistency in `src/ursatz.h`.\*\* One-line

&#x20;  change (`URSATZ\_H` → `MUSIC\_URSATZ\_H`), purely cosmetic, brings the last

&#x20;  stray header in line with the other 80. Not urgent, but essentially free and

&#x20;  removes the only inconsistency the convention has.

3\. \*\*Third, optional — de-duplicate the \~16-17 verbatim-repeated one-line field

&#x20;  comments\*\* (`"The Analyzer value pluggable..."` × 7, `"The owned base

&#x20;  Transformation..."` × 10). Low value on its own (16 lines out of 2,651), but if

&#x20;  you're already touching those files for something else, moving the shared

&#x20;  sentence into the base type's own header (`analyzer.h`, `transformation.h`) and

&#x20;  dropping the repeats is a nice, safe trim.

4\. \*\*Optional, moderate effort — add a table of contents to `docs/ARCHITECTURE.md`

&#x20;  and split the three lumped L6/L7/L8 sections into one heading per module\*\*, matching

&#x20;  the granularity already used for L0-L5. This is a documentation-navigation

&#x20;  improvement, not a size reduction (665 lines isn't large), so I'd only bother once

&#x20;  the previously-flagged status/gaps index is being added anyway — do them in the

&#x20;  same pass rather than two separate doc edits.

\*\*What I would explicitly leave alone:\*\*

&#x20;

\- \*\*The header doc-comment density overall.\*\* At \~85-90% load-bearing content across

&#x20; every module I sampled, this is exactly what good C header documentation should

&#x20; look like for a spec-driven project like this — spec citations, ownership rules,

&#x20; and explicitly flagged gaps are exactly the kind of thing that's expensive to

&#x20; reconstruct later and cheap to keep now. Trimming this "for size" would be a net

&#x20; loss disguised as cleanup. I would not touch it beyond item 3 above.

\- \*\*The `MUSIC\_` prefix.\*\* It's used correctly, minimally, and only where it does

&#x20; real work (header guards in a project with zero header-guard-collision risk

&#x20; otherwise). There is nothing to remove here — the premise that it might be

&#x20; over-applied doesn't hold up against the actual grep results.

\- \*\*The 400 NULL-guard blocks and 98 malloc/destroy patterns in `.c` files.\*\* These

&#x20; look repetitive in aggregate but are the standard, correct shape of a hand-rolled

&#x20; ADT-per-file C library with per-type error enums. Macro-izing them would trade a

&#x20; small line-count win for meaningfully worse debuggability (macro-expanded control

&#x20; flow, harder breakpoints/stack traces) and would touch 78 files for a

&#x20; purely-cosmetic gain — not worth the risk given this is explicitly a "don't touch

&#x20; correctness/architecture" pass.

\- \*\*`src/CMakeLists.txt`'s explicit file list.\*\* Looks long (82 lines) but an

&#x20; explicit list is the right call for a CMake project of this size; don't switch it

&#x20; to `GLOB`.

&#x20;

