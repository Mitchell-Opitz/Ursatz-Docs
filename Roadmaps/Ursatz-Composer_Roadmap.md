# Ursatz-Composer Roadmap: Design, Branches and Commits

**Written:** 2026-10-05. The repo does not exist yet. This document is the architect's proposed
design plus the build plan. Nothing here is Registry-authorized yet; each phase's types get
Registry rows before its first branch, as for any repo.

## What it is for

A composing partner, not a push-button song machine. Its job is to help its owner write good
music faster:
- "Here is a motive, grow it into a phrase."
- "Give me five continuations for this antecedent."
- "Plan a rounded binary form in D minor, 32 bars, and sketch it."
- "Make it sound like this corpus, with some of that one mixed in."

Everything it writes is seeded and reproducible, carries provenance, and can be locked, edited,
or regenerated piece by piece.

## Core design: one plan, filled from any direction

### The piece plan

The center of the Composer is a **piece plan**: the same tree the Analyzer writes for an
existing piece (Analyzer A4.4), so the Composer can learn from analyses and the Analyzer can
check the Composer's output with the same vocabulary.

```
Form            e.g. rounded binary, 32 bars, D minor
 └ Section      A, A', B, A''      key plan, cadence plan, thematic returns
    └ Phrase    8 bars             archetype (sentence, period, hybrid), cadence type
       └ Subphrase / Idea          presentation, continuation, basic idea, contrasting idea
          └ Motive                 a short pitch-and-rhythm cell, plus how it is transformed
```

Alongside the tree run **lanes**, time-aligned layers that every node owns a span of: meter
and hypermeter, key, harmony (Roman numerals with function), bass line, melody, inner voices
and texture.

Every node and every lane span is in one of three states:
- **Given:** fixed by the user (a motive you wrote, a form you chose). Never changed.
- **Constrained:** not written yet, but with requirements ("ends with a PAC", "8 bars",
  "uses motive m1 inverted").
- **Realized:** written by the Composer. Can be regenerated.

### One engine, three directions

There is a single loop, not separate top-down and bottom-up programs:

1. Pick the open node or span with the most constraints on it.
2. **Generate** candidates for it with an operator (below).
3. **Check** each candidate with framework rules: a hard-rule break rejects it, soft rules and
   the style model score it.
4. **Choose** one by score and seed (or show several to the user, in suggestion mode).
5. **Propagate**: the choice adds constraints to its parent, children and neighbors.
6. Repeat until nothing is open.

The direction comes only from **where the given material sits** and **which operators run**:

- **Top-down** (form → motives). The root is given. *Refine* operators split a node into
  children: form template → sections with a key and cadence plan → phrases with archetypes →
  ideas → motives, then lanes are filled phrase by phrase.
- **Bottom-up** (motives → form). A motive is given. *Grow* operators build a parent from
  children: motive → basic idea (motive plus a transformed restatement) → sentence or period →
  section → form, choosing the parent whose conventions best fit what exists.
- **Middle-out.** Give it one phrase, or a chord progression, or a bass line; grow up and
  refine down from there. This falls out of the same loop for free.

Both directions use the same rules the Analyzer applies, so "compose a sentence" and "is this a
sentence" can never disagree.

### Operators (where the theory comes from)

| Operator | Kind | Built from |
|---|---|---|
| Form templates (binary, rounded binary, ternary, theme and variations, loop forms for games) | Refine | `standard-form`, new templates |
| Phrase archetypes (sentence, period, hybrids, expansion) | Refine and grow | `standard-form` |
| Cadence and key plan | Refine | `standard-harmony` (cadences, tonicization, modulation) |
| Harmonic skeleton (Tb, PD, D, Te phrase model, prolongation, six-fours, sequences) | Fill harmony lane | `standard-harmony`, `scale-harmonization` |
| Galant schemas | Fill harmony and outer-voice lanes together | `galant-schemas` |
| Bass line | Fill bass lane | Rule of the octave, `standard-counterpoint` |
| Motivic development (transpose, invert, retrograde, fragment, augment, interval change) | Grow, fill melody | Library transforms, `IntervallicManipulation` |
| Melody over harmony (chord tones on strong beats, non-chord tones between) | Fill melody lane | `standard-tonality`, `NonChordTone`, `standard-counterpoint` tendencies |
| Voicing and accompaniment patterns | Fill inner lanes | `standard-voicing`, `standard-texture` |
| Rhythm and meter (hypermeter, syncopation, density curve) | Fill meter lane | `standard-meter`, `standard-rhythm` |

### Scoring

- **Hard rules** (framework constraints): parallel fifths, unresolved leading tone in a
  cadence, a cadence that isn't one. A candidate breaking one is never chosen.
- **Soft rules** (framework preferences): leap recovery, doubling preferences, climax placement.
- **Style score**: distance to a StyleModel learned from a corpus (Analyzer A7.2). Blending two
  models gives "whimsical like this, dark like that."
- **Self-check**: the finished phrase is run back through the Analyzer (A8.2). If the Analyzer
  doesn't hear a PAC where the plan wanted one, the phrase is rejected and regenerated.

### Where it lives

`Ursatz-Composer` is an L9 application: a C11 static library plus a CLI, with `external/ursatz`
and `external/ursatz-analyzer` as submodules. The kernel keeps the theory-neutral `Generator`
interface and base types (`Intent`, `GenerationPlan`, `CandidateSet`, `GenerationRecord`);
concrete, style-dependent generators live here, since generation must not require a framework
unless one is selected (Dependency spec). Natural-language intents are out of scope; intents
are structured (CLI flags or a JSON request).

### When to start (architect decision, 2026-10-05)

The earlier gate was "after Framework v1 verification." Revised: **phases C1 to C3 can start
once L5.1, L5.4 and L5.6 merge**, because writing rule-checked music needs the rules and a
renderer, not perception. Style learning and self-checking (C5, C8) still wait on the Analyzer.
This gets a usable sketching tool years sooner. Reflected in `Overview/Master_Roadmap.md`.

## How to read this

- Branch IDs (`C3.2`) are this repo's; `L`/`A` IDs refer to the other two roadmaps.
- Commit lines are title-only, `<type> - <Description>`.
- **Milestone M1** (end of C3) is the first thing worth using for real composing.

## Phase overview

| Phase | Goal | Needs |
|---|---|---|
| C1 Scaffold | Repo, plan model, I/O, rendering | L5.4, L5.6 |
| C2 Harmonic skeleton | Key + meter + length + cadence → progression and bass | L5.1, L5.2 |
| C3 Bottom-up phrases | Motive → idea → sentence or period, with melody | C2 |
| **M1** | **Motive in, 8-bar harmonized phrase out, as MIDI and MusicXML** | |
| C4 Texture and voicing | Accompaniment patterns, SATB and keyboard voicing | C3 |
| C5 Self-critique | Analyzer checks and ranks every candidate | A8.2 |
| C6 Top-down planning | Form templates, key and cadence plans, thematic returns | C3 |
| C7 Any direction | Middle-out, lock and regenerate, merging plans | C3, C6 |
| **M2** | **Plan a whole short piece from either end, then edit it node by node** | |
| C8 Style | Learned style models, blending, "material like this" | A7.2, L5.3, L5.5 |
| C9 Workflow | Suggestion commands, variations, export for a DAW | C7 |

---

## C1. Scaffold

### C1.1 `chore/scaffold-composer`
- `chore - Add CMake project with Ursatz and Ursatz-Analyzer submodules`
- `chore - Add CI workflow with submodule checkout`
- `docs - Add README with purpose and build steps`

### C1.2 `feat/piece-plan-model`
- `feat - Add PiecePlan with form, section, phrase, subphrase, idea and motive nodes`
- `feat - Add time-aligned lanes for meter, key, harmony, bass, melody and inner voices`
- `feat - Track given, constrained and realized state per node and lane span`
- `feat - Attach constraints to nodes and spans`
- `test - Build and walk a plan for an 8-bar sentence`

### C1.3 `feat/plan-json-io`
- `feat - Read and write a plan as JSON using the analysis document vocabulary`
- `feat - Load an analysis document as a fully realized plan`
- `test - Round-trip a plan through JSON`
- `docs - Document the plan format`

### C1.4 `feat/render-plan`
- `feat - Render realized lanes to NoteEvents`
- `feat - Export a plan to MIDI and MusicXML through Ursatz`
- `test - Render a hand-written plan and re-import it`

### C1.5 `feat/fill-loop`
The engine, with no operators yet.
- `feat - Pick the most constrained open node or span`
- `feat - Generate, check, choose and propagate through registered operators`
- `feat - Seed every choice and write a GenerationRecord`
- `feat - Add a suggestion mode that returns the top N candidates instead of choosing`
- `test - Fill loop is reproducible from its seed`
- `docs - Document the fill loop and the operator interface`

---

## C2. Harmonic skeleton

### C2.1 `feat/phrase-model-progressions`
- `feat - Generate Tb, PD, D, Te progressions for a phrase length and cadence type`
- `feat - Place chord changes on the harmonic rhythm of the meter`
- `feat - Check progressions with standard-harmony rules`
- `test - Every generated progression ends in its requested cadence`

### C2.2 `feat/prolongation-and-embellishment`
- `feat - Prolong tonic and dominant with neighbor, passing and six-four chords`
- `feat - Add strong predominants and predominant sevenths`
- `feat - Add applied chords for tonicization`
- `test - Embellished progressions stay valid under standard-harmony`

### C2.3 `feat/bass-line`
- `feat - Realize a bass line from the progression and inversions`
- `feat - Harmonize stepwise bass with the rule of the octave`
- `feat - Check the bass with counterpoint melodic rules`
- `test - Bass lines avoid forbidden leaps and outline cadences correctly`

### C2.4 `feat/schema-and-sequence-skeletons`
- `feat - Offer galant schemas as phrase openings, continuations and cadences`
- `feat - Offer harmonic sequences for continuations`
- `test - Schema skeletons match their prototypes`

### C2.5 `feat/cli-skeleton`
- `feat - Add "ursatz-composer progression --key --meter --bars --cadence"`
- `docs - Document the progression command`

---

## C3. Bottom-up phrases

### C3.1 `feat/motive-input`
- `feat - Accept a motive as MIDI, MusicXML or note list`
- `feat - Analyze the motive's intervals, rhythm and implied harmony`
- `test - Motive input from each format`

### C3.2 `feat/motivic-development`
- `feat - Generate restatements by transposition, inversion and interval change`
- `feat - Generate fragments and liquidations for continuations`
- `feat - Generate rhythmic augmentation and diminution`
- `feat - Score developments by recognizability (shared contour and rhythm)`
- `test - Every development is detected as a relative by motive analysis`

### C3.3 `feat/melody-over-harmony`
- `feat - Place motive notes as chord tones on strong beats`
- `feat - Fill between chord tones with passing and neighbor tones`
- `feat - Shape phrase contour with one climax and recovered leaps`
- `feat - Check melody against bass with counterpoint harmonic rules`
- `test - Melodies keep tendency tones resolved at cadences`

### C3.4 `feat/grow-sentence`
- `feat - Grow a basic idea from a motive and one restatement`
- `feat - Grow a presentation from a basic idea and its repetition`
- `feat - Grow a continuation with fragmentation and harmonic acceleration`
- `feat - Close the sentence with its planned cadence`
- `test - Analyzer identifies every generated sentence as a sentence`

### C3.5 `feat/grow-period`
- `feat - Grow an antecedent ending in a weak cadence`
- `feat - Grow a consequent that restates the basic idea and ends stronger`
- `feat - Grow hybrid themes from the nine beginning and ending pairs`
- `test - Analyzer identifies generated periods and hybrids correctly`

### C3.6 `feat/cli-grow`
- `feat - Add "ursatz-composer grow <motive> --into sentence|period|hybrid"`
- `feat - Add --suggest N to return several phrases`
- `docs - Document the grow command`

**Milestone M1:** give it a one-bar motive, a key and a meter; get back several 8-bar
harmonized phrases with bass and melody, as MIDI and MusicXML.

---

## C4. Texture and voicing

### C4.1 `feat/accompaniment-patterns`
- `feat - Realize block, arpeggiated, Alberti and oom-pah accompaniments`
- `feat - Choose accompaniment by texture target`
- `test - Accompaniment notes stay inside the chord and range`

### C4.2 `feat/four-part-voicing`
- `feat - Voice progressions in SATB with standard-voicing rules`
- `feat - Connect voicings with smooth voice leading`
- `test - Generated chorales pass voice-leading analysis`

### C4.3 `feat/two-part-counterpoint`
- `feat - Write a counter-melody against a given line in a chosen species`
- `test - Generated counterpoint passes standard-counterpoint`

---

## C5. Self-critique

### C5.1 `feat/analyzer-check-loop` (needs A8.2)
- `feat - Analyze each realized phrase in memory`
- `feat - Reject phrases whose analysis contradicts the plan`
- `feat - Rank surviving candidates by analysis agreement and soft-rule score`
- `test - Planned cadences are always heard as planned`

### C5.2 `feat/explain-choices`
- `feat - Report why each candidate was chosen or rejected`
- `docs - Document the explanation report`

---

## C6. Top-down planning

### C6.1 `feat/form-templates`
- `feat - Add binary, rounded binary and ternary templates`
- `feat - Add theme and variations`
- `feat - Add loop and layered forms for game and media music`
- `test - Each template refines into sections with a cadence plan`

### C6.2 `feat/key-and-cadence-plans`
- `feat - Plan section keys with closely related modulations`
- `feat - Plan cadence strength across sections`
- `test - Plans follow standard-harmony key-change rules`

### C6.3 `feat/thematic-returns`
- `feat - Reuse a realized phrase where the form returns its theme`
- `feat - Vary a returning theme (ornament, reharmonize, re-register)`
- `test - Analyzer detects every planned thematic return`

### C6.4 `feat/cli-plan`
- `feat - Add "ursatz-composer plan --form --key --meter --bars"`
- `feat - Add "ursatz-composer fill <plan>" to realize a plan`
- `docs - Document the plan and fill commands`

---

## C7. Any direction

### C7.1 `feat/middle-out`
- `feat - Accept a given phrase, progression or bass line anywhere in a plan`
- `feat - Grow parents and refine children around given material`
- `test - A given middle phrase ends up inside a coherent form`

### C7.2 `feat/lock-and-regenerate`
- `feat - Lock any node or span as given`
- `feat - Regenerate one node and repropagate only what depends on it`
- `test - Regenerating a phrase leaves locked material untouched`

**Milestone M2:** plan a whole short piece from either end, then lock what you like and
regenerate the rest.

---

## C8. Style

### C8.1 `feat/style-models` (needs A7.2, L5.3)
- `feat - Load a StyleModel produced by the Analyzer's corpus mode`
- `feat - Score candidates by distance to the style model`
- `test - Phrases generated under a style score closer to it than unstyled ones`

### C8.2 `feat/style-blending`
- `feat - Blend two style models with a weight per dimension (harmony, rhythm, melody, form)`
- `feat - Add --style and --blend options`
- `test - Blend weight moves output between the two styles`

### C8.3 `feat/seed-from-corpus` (needs L5.5)
- `feat - Start from material similar to a given passage in a corpus`
- `docs - Document style and seeding options`

---

## C9. Workflow

### C9.1 `feat/variations-and-alternatives`
- `feat - Generate variations of a realized phrase`
- `feat - Generate alternative harmonizations of a given melody`

### C9.2 `feat/daw-friendly-export`
- `feat - Export one MIDI track per lane with clear names`
- `feat - Export tempo and markers for sections`

### C9.3 GUI integration (tracked in Ursatz-GUI)
A Composer tool module in the GUI: plan view, score preview, lock/regenerate buttons. Built
only after M2, against the Composer library, not a copy of it.
