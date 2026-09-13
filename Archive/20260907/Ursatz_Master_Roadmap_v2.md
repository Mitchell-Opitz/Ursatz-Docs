\# Master Roadmap v2 — Ursatz Ecosystem

&#x20;

Status as of: Phase 1 complete, Phase 2 starting. Project started

Sept 2 (code side). This supersedes v1 — v1's phase breakdown was

directionally correct; this version reflects what actually happened,

what was learned, and adds Phase 2.5, which wasn't separated out in v1.

&#x20;

This document is written to be self-sufficient for anyone (including a

future you, or a fresh Claude Code/Claude conversation) picking this

project up without prior context.

&#x20;

\---

&#x20;

\## What this project is

&#x20;

\*\*Ursatz\*\* — a general-purpose, theory-neutral computational music

platform, written in C. Core principle: no single music theory,

notation system, or generation algorithm gets baked into the kernel.

Facts about music (notes, timing, structure) are kept strictly separate

from \*interpretations\* of those facts (what chord is this, what key is

this), which are only ever produced under an explicit, named, swappable

"Theory Framework" — never assumed by the kernel itself.

&#x20;

\*\*Long-term goal\*\*: a system that can analyze real music and eventually

generate original music, encoding real music-theory knowledge (the

project owner's own, at an advanced level) as software — not a

from-scratch ML model, but a rule/preference-based symbolic system,

closer to the algorithmic-composition tradition than deep learning.

&#x20;

\*\*Two repositories exist / are planned\*\*, deliberately separated:

\- `Ursatz` — the library. Theory-neutral kernel + pluggable frameworks.

\- `ursatz-analyzer` — first consumer app (CLI). Depends on Ursatz.

\- (planned) a GUI consumer app — Phase 2.

\- (planned) a generation engine consumer app — Phase 3.

\---

&#x20;

\## Process notes — how this has been run (read this before continuing)

&#x20;

This matters as much as the phase list. The discipline below is \*why\*

things have gone well, and should keep being followed:

&#x20;

1\. \*\*Registry-first for any new type.\*\* Nothing gets implemented in

&#x20;  Ursatz without first being recorded in the System Registry (a

&#x20;  separate document, not in the repo). Claude Code never has direct

&#x20;  access to architecture docs — the project owner is the intentional

&#x20;  checkpoint between the architecture conversation and implementation.

2\. \*\*Layer-by-layer, narrow scope per task.\*\* Each Claude Code task is

&#x20;  handed one scoped piece of work (one branch's worth), not a whole

&#x20;  phase at once. This kept every task reviewable and kept mistakes

&#x20;  small and cheap to catch.

3\. \*\*Claude Code flags judgment calls before implementing; architect

&#x20;  (this conversation) reviews and approves/adjusts before code is

&#x20;  written.\*\* This caught several real design issues before they became

&#x20;  expensive:

&#x20;  - The circular-dependency problem when wiring a Theory Framework into

&#x20;    analysis (resolved by splitting build targets).

&#x20;  - The realization that `constraint\_evaluate` (L7) is a deliberate

&#x20;    stub and real matching logic belongs in the framework module

&#x20;    itself, not the kernel.

&#x20;  - The one-directional `PitchIdentity` conversion design, preserved

&#x20;    correctly under pressure to "just make it work."

4\. \*\*When something breaks on real-world data, diagnose root cause

&#x20;  before proposing a fix — and prefer fixing the actual generalizable

&#x20;  bug over patching the specific symptom.\*\* The key-detection bug

&#x20;  (exact-match-required-across-whole-piece, which fails on almost all

&#x20;  real music) is the clearest example: the fix generalized the same

&#x20;  tolerance already used at chord-level to key-level, rather than

&#x20;  special-casing the one failing file.

5\. \*\*Deliberate, disclosed scope cuts, not silent gaps.\*\* Every "not

&#x20;  built yet" is documented at the point of the gap, with a reason, so

&#x20;  future audits don't have to rediscover it. Several stale-comment

&#x20;  corrections happened along the way as things got resolved — worth

&#x20;  continuing to catch stale docs promptly, but not worth interrupting

&#x20;  momentum over minor cases (a batch documentation cleanup pass is

&#x20;  fine to defer, as was explicitly decided once).

6\. \*\*Grading reality check, for calibration going forward:\*\* the

&#x20;  \*process\* has consistently been excellent — disciplined, tested,

&#x20;  honest about gaps. The \*percentage of the ultimate vision complete\*

&#x20;  is a separate, much lower number (roughly 20-25% as of Phase 1) and

&#x20;  should stay separate in anyone's head — a good process moving

&#x20;  correctly is not the same claim as "almost done."

\---

&#x20;

\## Where things actually stand (end of Phase 1)

&#x20;

\*\*Ursatz library:\*\*

\- L0-L8-Transform: fully implemented, verified, tested, valgrind-clean.

\- Two originally-open architectural issues resolved: §7.1

&#x20; (Interpretation↔Framework ordering, resolved via `FrameworkReference`)

&#x20; and §7.2 (Theory↔Corpus ordering, confirmed already correctly handled

&#x20; via opaque pre-computed values).

\- A real, working, minimal Theory Framework exists:

&#x20; `frameworks/common-practice-minimal/` — identifies the 7 basic

&#x20; diatonic triads (I-vii°) and major/natural-minor keys, from real note

&#x20; data, deterministically (single answer, no confidence weighting yet).

\- `music-io` module exists with a real, deliberately minimal

&#x20; `MIDIImporter` — parses SMF type 0/1, note-on/off only, fixed 120 BPM

&#x20; assumption, flattens multi-track files, fails cleanly on malformed

&#x20; input rather than guessing.

\- L8-Analysis: `ChordIdentification` and `KeyEstimation` are real and

&#x20; wired to the framework above (not the other 6 analyzers — Motif,

&#x20; Phrase Segmentation, Voice Leading, Scale Analysis, Cadence Detection

&#x20; are all still deliberately unbuilt).

\- L4 (containers: Work, Section, Movement, etc.), L9 persistence,

&#x20; L9 corpus, L8-Generation: still not started, all deliberately

&#x20; deferred, not blocking anything done so far.

\*\*ursatz-analyzer (CLI):\*\*

\- Real, working end-to-end pipeline: `analyzer path/to/file.mid` →

&#x20; parses the file, runs chord/key analysis, writes a Markdown report,

&#x20; prints a confirmation.

\- Verified against real, unmodified commercial music (Pachelbel's Canon

&#x20; in D), not just synthetic test fixtures.

\- One real bug found and fixed during this verification: global key

&#x20; estimation originally required a piece's entire pitch-class content to

&#x20; be an \*exact\* 7-class match — any single non-diatonic tone (extremely

&#x20; common in real music) broke it. Fixed to use containment matching

&#x20; (all 7 required, extras tolerated), mirroring logic already used at

&#x20; the individual-chord level.

\*\*Known, disclosed, NOT yet fixed (candidate for Phase 2.5, see below):\*\*

\- Chord-region matching still has the same rigidity the key-level bug

&#x20; had, one layer down — a sonority likely requires an exact match with

&#x20; no tolerance for extra simultaneous non-chord tones (e.g. a passing

&#x20; tone sounding against a sustained triad). This is why real files still

&#x20; show many "no match" chord regions even after the key-level fix.

&#x20; Diagnosed as very likely the same category of bug; not yet confirmed

&#x20; or fixed. \*\*First item to pick up in Phase 2.5.\*\*

\---

&#x20;

\## PHASE 2 — Bare-bones Library GUI (current phase)

&#x20;

Unchanged in shape from v1, still the right next step: get real

persistence into Ursatz, then a minimal GUI on top.

&#x20;

\### Branch (Ursatz repo): `feat/persistence-minimal`

\- Minimal storage for NoteEvent sequences + their Chord/Key analysis

&#x20; results + Provenance. Doesn't need to satisfy the full L9

&#x20; `EntityRepository` spec, just enough for a personal-scale tool.

&#x20; SQLite is the presumed default unless a build-environment reason rules

&#x20; it out — confirm with Claude Code rather than assuming.

\### Branch (new repo, e.g. `ursatz-library-gui`): `feat/gui-scaffold`

\- GUI framework choice is a real decision — do not let Claude Code

&#x20; default silently. Decide deliberately (desktop framework vs. local

&#x20; web UI) before scaffolding.

\- Persistence/query access must be built as a shared service the GUI calls, not as code owned by the analysis-viewer tool — so future tools (e.g. Generator) can call the same service without going through the analyzer.

\### Branch: `feat/add-and-analyze-flow`

\- Add MIDI file → runs importer + analysis → saves via persistence.

\- List view of saved pieces.

\- "Re-run analysis" — re-executes against a saved piece without

&#x20; re-importing, useful once the framework or bug fixes improve results

&#x20; (directly useful once Phase 2.5's chord-matching fix lands).

\### Branch: `feat/provenance-view`

\- Detail view per piece, showing analysis results and their Provenance

&#x20; (which importer/analyzer produced them, when) — first real use of the

&#x20; Provenance work built all the way back at L1/L6.

\---

&#x20;

\## PHASE 2.5 — Analysis Engine Refinement (new in v2, was implicit in v1)

&#x20;

Not a fixed scope like other phases — this is an ongoing "make the

analysis actually good on real music" effort, picked up once Phase 2's

GUI/persistence loop exists to make iteration fast (add file → see

result → tweak → re-run, without re-running a CLI by hand each time).

&#x20;

\*\*Known first item:\*\* fix chord-region matching's likely exact-match

rigidity, same pattern/fix shape as the key-detection bug — containment/

best-fit matching instead of requiring an exact set with nothing extra.

&#x20;

\*\*Expected shape of this phase, based on what's been learned so far:\*\*

\- More real-world bugs of the same "works on synthetic data, breaks on

&#x20; real data" category should be expected — this isn't a one-off, it's a

&#x20; pattern with real music (real recordings/arrangements are messier than

&#x20; clean test fixtures). Budget time for this rather than treating each

&#x20; discovery as a surprise.

\- This is also where \*\*multi-candidate, confidence-weighted analysis\*\*

&#x20; belongs (discussed but explicitly deferred during Phase 0): instead of

&#x20; one deterministic answer, surface multiple plausible interpretations

&#x20; with weights (e.g. "65% vii°, 25% V, 10% something else") for

&#x20; genuinely ambiguous sonorities. This maps onto the already-existing

&#x20; `Candidate` type and `Interpretation`'s uncertainty field — both exist

&#x20; as concepts, unused so far. The database/GUI (Phase 2) is what stores

&#x20; and displays these; the analyzer is what needs to start producing them.

\- Expanding the Theory Framework itself (more chord types, cadence

&#x20; detection, possibly a second framework) likely belongs here too,

&#x20; time permitting, before Phase 3 — Phase 3's generation quality

&#x20; depends directly on how good this analysis/theory foundation is.

\*\*No fixed exit criteria yet\*\* — revisit and scope this phase's end

point once you're inside it and have a feel for how much real-world

messiness keeps surfacing.

&#x20;

\---

&#x20;

\## PHASE 3 — Generative Composition Engine (unchanged in shape, more detail on the real gap)

&#x20;

Still correctly gated behind Phases 1-2.5 being real, for a specific

reason worth restating clearly since it came up directly:

&#x20;

\*\*The real gap Phase 3 has to close isn't more rules — it's a ranking/

preference layer.\*\* Constraint-checking (does this candidate satisfy

rule X) is a yes/no filter; it can reject bad options but can't produce

or rank good ones. Generation needs a genuinely different kind of logic:

given many rule-satisfying candidates, which one is actually musically

preferable. The Registry already has the right concepts reserved for

this (`Preference`, `Heuristic`, `CandidateEvaluator`) — none built yet.

&#x20;

\*\*Expect an empirical, listen-and-adjust loop, not a pure design

process.\*\* Music-theory knowledge splits into hard rules (rare, easy to

encode) and soft preferences/taste (most of it, harder to encode,

often only validated by actually listening to output and adjusting

weights). This is not fully solvable by reasoning about it in advance —

budget real iteration time once generation exists, the same way a

composition teacher's ear develops over years, not by fixing one rule

at a time in isolation.

&#x20;

Depends on, not yet built:

\- Full L8-Generation (currently stubbed/conceptual)

\- A more complete Theory Framework than the current minimal one

\- Real `StatisticalModel`/`StyleModel` (currently opaque-value-only) —

&#x20; needs actual corpus data, which Phase 2's stored library of analyzed

&#x20; pieces starts to provide

\- L9 Corpus module (not started)

Still not broken down step-by-step — genuinely premature until Phase

2/2.5 produce a real library of analyzed pieces and a proven-decent

analysis engine to build a StyleModel and preference layer against.

&#x20;

\---

&#x20;

\## Summary table (updated)

&#x20;

| Phase | Repo | Status | Depends on |

|---|---|---|---|

| 0 | Ursatz | Done | — |

| 1 | ursatz-analyzer | Done | Phase 0 |

| 2 | Ursatz (persistence) + new GUI repo | In progress | Phase 1 |

| 2.5 | Ursatz + ursatz-analyzer (ongoing refinement) | Not started | Phase 2 (for iteration speed) |

| 3 | new repo (generation) | Not started | Phases 1-2.5 in real use |

&#x20;

Each branch is still sized to be its own Claude Code task, same

architect-review-then-implement pattern used throughout. Continue it.

&#x20;

