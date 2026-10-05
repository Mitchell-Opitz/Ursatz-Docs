# Ursatz (Library) Roadmap: Branches and Commits

**Written:** 2026-10-05, against Ursatz `main` at `090695c` (PR #97). Planned work, not status:
for what is built, see `Ursatz-Library/Status.md`. Every branch goes through the Registry
first (`Ursatz-Library/Dependency_Layer_Specification.md`) and follows the perception path
rule: no new rule without a way for analysis to reach it.

## How to read this

- Branch IDs (`L1.2`) are for cross-referencing from the Analyzer and Composer roadmaps.
  Branch names follow the repo's `<type>/<kebab-description>` convention.
- Commit lines are title-only, in the repo's `<type> - <Description>` style.
- "Depends on" lists branches that must merge first. Within a phase, branches run in order
  unless marked parallel.
- "Done when" is the merge gate, on top of green CI.

## Phase overview

| Phase | Goal | Unblocks |
|---|---|---|
| L1 Housekeeping | CI runs everything; known architecture debt cleared | Trust in every later branch |
| L2 Perception | MIDI and MusicXML in, labeled musical facts out (meter, chords, lines, keys) | Analyzer A2 onward |
| L3 Framework adapters | The existing rule libraries produce Interpretations from real music | Analyzer A3/A4, Composer evaluation |
| L4 Verification | Accuracy measured against annotated datasets | Confidence to build the Composer on analysis output |
| L5 Generation enablers | Real constraint and preference evaluation, style statistics, rendering | Composer C2 onward |
| L6 Breadth | More styles, more input formats, as needed | Later |

---

## L1. Housekeeping

### L1.1 `ci/run-framework-tests`
Run all framework tests from the root build and in CI.
- `ci - Add framework test directories to the root build when testing is enabled`
- `fix - Allow common-practice-minimal's own seventh-chord headers in its architecture test`
- `ci - Run framework tests in the CI workflow`
- `docs - Remove the framework test workaround from BUILDING.md`

Done when: root `ctest` runs 468 tests, all pass.

### L1.2 `refactor/remove-unused-vocabulary-enums`
Remove the three vocabulary enums no code uses.
- `refactor - Remove SubjectPart, SubjectCharacter and SchemaSense value types`
- `docs - Drop removed enums from semantics module docs`

### L1.3 `refactor/cadence-vocabulary-owner`
Make `CadenceType` the one cadence vocabulary and `standard-harmony` the one strength order.
- `feat - Add plagal and deceptive to CadenceType`
- `feat - Expose standard-harmony cadence strength ordering`
- `refactor - Classify cadences as CadenceType in cadence classification`
- `refactor - Rank period cadences by standard-harmony strength`
- `refactor - Rank standard-form cadences by standard-harmony strength`
- `test - Update cadence classification, period and form tests for CadenceType`
- `docs - Document CadenceType as the cadence vocabulary owner`

Done when: no cadence string labels or private strength tables remain.

### L1.4 `feat/midi-key-signature-value`
Make the MIDI key-signature scanner produce the `KeySignature` value.
- `feat - Build a KeySignature from a scanned MIDI key signature event`
- `feat - Report every key signature event with its position, not only the first`
- `test - Add key signature change fixtures and tests`
- `docs - Document key signature events in io module docs`

### L1.5 `chore/phrase-expansion-decision`
Merge or drop `feat/phrase-expansion` under the perception path rule. Recommended: merge, since
phrase expansion is needed by Analyzer form reports (A4.3) and Composer phrase building (C3).
Rebase `feat/phrase-expansion` onto `main` and open it as its own PR rather than adding commits
here. If dropped instead, delete the branch and add one commit:
- `docs - Note phrase expansion as planned work under the form adapter (L3.6)`

---

## L2. Perception (MIDI and notation in, labeled facts out)

### L2.1 `feat/midi-time-signatures`
Read meter from MIDI.
- `feat - Decode time signature meta-events in the MIDI event decoder`
- `feat - Add MeterMap mapping time spans to TimeSignature`
- `feat - Return a MeterMap from the MIDI importer`
- `feat - Resolve Measure meter regions against MeterMap`
- `test - Add 3/4, 6/8 and 12/8 import fixtures and measure index tests`
- `test - Check Field Nocturne No. 1 imports as 12/8`
- `docs - Document meter import and MeterMap in time and io module docs`

Done when: measure numbers match the score for all 5 reference pieces.

### L2.2 `feat/midi-tempo-map`
Read tempo, so performance timing is recoverable and exports carry tempo.
- `feat - Decode set-tempo meta-events in the MIDI event decoder`
- `feat - Add TempoMap mapping MusicalTime to seconds as exact rationals`
- `feat - Return a TempoMap from the MIDI importer`
- `test - Add tempo change fixtures and tests`
- `docs - Document tempo import in io module docs`

### L2.3 `feat/metric-position`
Turn positions into bar, beat and beat strength.
- `feat - Add MetricalPosition value (measure, beat, subdivision)`
- `feat - Derive metrical position from MusicalTime and MeterMap`
- `feat - Assign BeatStrength from metrical position under standard-meter`
- `test - Add metrical position and beat strength tests for simple and compound meters`
- `docs - Document metrical position in time module docs`

### L2.4 `feat/adaptive-sonority-segmentation`
Replace the fixed quarter-note grid with meter-aware, change-point segmentation.
- `feat - Segment sonorities on the beat grid of the current meter`
- `feat - Merge adjacent beats with unchanged pitch-class content`
- `feat - Split a beat when its bass changes`
- `test - Compare segment boundaries against hand-marked chord changes in Bach 114`
- `docs - Document adaptive segmentation and its limits in semantics module docs`

### L2.5 `feat/weighted-chord-identification`
Fix "Problem B": choose chords by weighted fit, treating extra tones as non-chord tones.
- `feat - Weight sonority pitch classes by duration and beat strength`
- `feat - Score triad and seventh candidates by weighted chord-tone coverage`
- `feat - Label leftover tones with NonChordTone by approach and departure`
- `feat - Report the best and runner-up chord with uncertainty`
- `test - Field Nocturne No. 1 progression matches the known I-V-I sequence`
- `test - Report chord identification coverage for the five reference pieces`
- `docs - Replace the subset-match note with weighted identification in analysis docs`

Done when: Field 1 resolves at least 90% of chord segments to the right harmony.

### L2.6 `feat/outer-voice-lines`
Derive soprano and bass lines, so PAC/IAC and line rules get real inputs.
- `feat - Extract the highest and lowest sounding line per segment`
- `feat - Smooth outer lines across short gaps and register leaps`
- `feat - Expose scale degree of soprano and bass per chord segment`
- `test - Outer lines match the written melody and bass of Bach 114 and Mozart 545`
- `docs - Document outer-voice extraction in analysis docs`

### L2.7 `feat/voice-separation`
Separate inner voices for multi-voice textures (needed for voice leading and counterpoint).
- `feat - Assign notes to voices by pitch proximity and continuity`
- `feat - Keep voice assignment stable across rests`
- `test - Voice separation on a four-part chorale fixture`
- `docs - Document voice separation and its limits`

**Decision gate before L2.7:** if rule-based separation can't reach usable accuracy on the
corpus, allow a statistical method here, reported as an `Observation` with `Uncertainty`, never
as a Rule (Law 12). Record the decision in `Reconciliation_Log.md`.

### L2.8 `feat/local-key-regions`
Track keys over time: tonicization, modulation, pivot chords.
- `feat - Estimate key per phrase-length window`
- `feat - Detect key changes and their extent with KeyChangeExtent`
- `feat - Find pivot chords at key change boundaries`
- `feat - Use key signature change events as authoritative boundaries when present`
- `test - Key regions for Clementi 36 and Mozart 545`
- `docs - Document local key regions in analysis docs`

### L2.9 `feat/musicxml-import`
Read sheet music, not just MIDI. Notated input gives spelling, meter, voices and staves directly.
- `feat - Add MusicXML reader for score-partwise notes, rests and ties`
- `feat - Import measures, time signatures and key signatures from MusicXML`
- `feat - Import parts, staves and voices from MusicXML`
- `feat - Import tempo and dynamics markings from MusicXML`
- `test - Round-trip the five reference pieces through MusicXML export and import`
- `docs - Add MusicXML import to io module docs`

---

## L3. Framework adapters (rules meet real music)

Each adapter turns L2 facts into a framework's inputs and returns `Interpretation`s, following
the durable claim pattern. Adapters live in their framework (`frameworks/<name>/analysis/`), not
in `src/analysis/`.

### L3.1 `feat/roman-numeral-analysis`
- `feat - Build standard-tonality chord input from identified chord segments`
- `feat - Produce Roman numeral Interpretations per segment in its local key`
- `feat - Label applied chords with AppliedChordKind`
- `test - Roman numerals for Bach 114 match a hand analysis`
- `docs - Document the Roman numeral adapter in standard-tonality`

### L3.2 `feat/harmonic-function-analysis`
- `feat - Assign HarmonicFunction to each Roman numeral`
- `feat - Label phrases with PhraseModelLabel (Tb, PD, D, Te)`
- `feat - Identify cadential, passing and neighbor six-fours`
- `feat - Identify prolongations of tonic and dominant`
- `test - Phrase model labels for Mozart 545 exposition`
- `docs - Document the harmony adapter in standard-harmony`

### L3.3 `feat/cadence-analysis-from-music`
- `feat - Classify cadences from Roman numerals and soprano degree`
- `feat - Distinguish PAC from IAC using the soprano line`
- `feat - Detect half, deceptive, plagal and subverted cadences`
- `test - Cadence types for all five reference pieces against a hand analysis`
- `docs - Replace "authentic (unclassified)" note in analysis docs`

### L3.4 `feat/voice-leading-analysis`
- `feat - Classify motion between voice pairs with MotionType`
- `feat - Check parallel and direct fifths and octaves across separated voices`
- `feat - Check seventh and leading tone resolution`
- `test - Voice leading report on a chorale fixture`
- `docs - Document real voice leading analysis`

### L3.5 `feat/melody-analysis`
- `feat - Compute melodic intervals and contour from outer lines`
- `feat - Find the climax and evaluate leap recovery`
- `feat - Label non-chord tones in the melody`
- `test - Melody analysis of Bach 114`
- `docs - Document melody analysis`

### L3.6 `feat/phrase-form-analysis`
- `feat - Segment phrases from cadences and metrical position`
- `feat - Segment ideas and subphrases with standard-form length conventions`
- `feat - Identify sentence, period and hybrid themes from segmented phrases`
- `feat - Detect phrase expansion and contraction`
- `test - Phrase form of the five reference pieces against a hand analysis`
- `docs - Document form analysis in standard-form`

### L3.7 `feat/motive-analysis`
- `feat - Detect motives by interval and rhythm, transposition-invariant`
- `feat - Match inversion, retrograde and augmentation of a motive`
- `feat - Match fragmented motives in continuations`
- `test - Motive families in Mozart 545`
- `docs - Document motive analysis and its transformation matching`

### L3.8 `feat/rhythm-meter-analysis`
- `feat - Detect syncopation and hemiola against the meter`
- `feat - Identify hypermeter from phrase and cadence placement`
- `feat - Profile rhythmic density per phrase`
- `test - Rhythm analysis of Field 5`
- `docs - Document rhythm and meter analysis`

### L3.9 `feat/texture-analysis`
- `feat - Measure texture features from separated voices`
- `feat - Classify texture per passage with standard-texture`
- `test - Texture changes in Field 1`
- `docs - Document texture analysis`

### L3.10 `feat/schema-sequence-analysis` (after L3.1-L3.5)
- `feat - Match galant schema prototypes against outer-voice degrees`
- `feat - Detect harmonic sequences by kind`
- `test - Schema and sequence detection on a galant fixture`
- `docs - Document schema and sequence detection`

### L3.11 `refactor/vocabulary-enum-relocation` (parallel, opportunistic)
Move the 24 single-framework enums into their frameworks as each framework is touched by L3.
One commit per enum group, e.g. `refactor - Move HarmonicFunction into standard-harmony`.

---

## L4. Verification

### L4.1 `test/annotated-corpus-harness`
Measure accuracy against published annotations instead of eyeballing.
- `test - Add a loader for Roman numeral annotation files`
- `test - Import a small annotated corpus subset as fixtures`
- `test - Report chord, key and cadence accuracy per piece`
- `ci - Fail the build when accuracy drops below the recorded baseline`
- `docs - Document the accuracy baseline and how to raise it`

### L4.2 `test/framework-v1-corpus-verification`
Close Phase 2.5 for real.
- `test - Assert keys, cadences and phrase form for all five reference pieces`
- `test - Assert thematic returns for Mozart 545 and Clementi 36`
- `docs - Record Framework v1 verification results`

---

## L5. Generation enablers

### L5.1 `feat/framework-constraint-evaluation`
Make `constraint_evaluate` real by letting a framework register how its rules evaluate.
- `feat - Add an evaluation callback to Constraint`
- `feat - Evaluate constraints through their framework callback`
- `feat - Register rule evaluation callbacks in standard-harmony and standard-counterpoint`
- `test - Constraint evaluation against framework rules`
- `docs - Retire the constraint_evaluate stub note`

### L5.2 `feat/preference-ranking`
- `feat - Rank candidates by weighted soft-rule satisfaction`
- `feat - Combine preference scores with exact-rational weights`
- `test - Preference ranking never rejects a candidate`
- `docs - Retire the preference_rank stub note`

### L5.3 `feat/corpus-style-statistics`
Learn style tendencies from analyzed music (as Tendency, never as Rule).
- `feat - Count chord progressions, cadence types and phrase forms across a corpus`
- `feat - Count melodic intervals, rhythms and motive transformations across a corpus`
- `feat - Build a StyleModel from corpus statistics`
- `feat - Blend two StyleModels with a weight`
- `test - Style statistics for the reference corpus`
- `docs - Document style statistics and blending`

### L5.4 `feat/generator-interface`
- `feat - Implement Generator dispatch over registered generators`
- `feat - Record a GenerationRecord with seed and versions for every run`
- `test - Generation is reproducible from its seed`
- `docs - Document the Generator contract for consumer apps`

### L5.5 `feat/similarity-query`
Let the composer find "material like this."
- `feat - Add SimilarityModel over interval, rhythm and contour dimensions`
- `feat - Search a corpus for similar motives and progressions`
- `test - Similarity search on the reference corpus`
- `docs - Document similarity search`

### L5.6 `feat/midi-renderer`
- `feat - Render NoteEvents to a type 1 MIDI file`
- `feat - Write tempo, meter and key signature events`
- `test - Round-trip MIDI import and render`
- `docs - Document the MIDI renderer`

### L5.7 `feat/musicxml-export-real-meter`
- `feat - Export measures from MeterMap instead of a synthetic 4/4 grid`
- `feat - Export multiple parts and staves`
- `feat - Export key signatures, tempo and dynamics`
- `test - Export the reference pieces with their real meters`
- `docs - Remove the synthetic meter note`

### L5.8 `feat/relationship-structure-repositories`
- `feat - Add RelationshipRepository`
- `feat - Add StructureRepository for motif, cadence and phrase shells`
- `test - Persist and reload a full analysis`
- `docs - Document the new repositories`

---

## L6. Breadth (pulled by need, not scheduled)

Each needs a perception path first. Candidates, in rough order of value for a composer of game
and media music: chromatic harmony (mixture, Neapolitan, augmented sixths), jazz/pop chord
symbols and extended harmony, modal and post-tonal frameworks, orchestration and instrument
ranges (`Instrument` already exists), loop and ostinato forms for game music, ABC import.
