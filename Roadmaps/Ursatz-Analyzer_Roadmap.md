# Ursatz-Analyzer Roadmap: Branches and Commits

**Written:** 2026-10-05, against Ursatz-Analyzer `main` at `718c86a` (PR #5) and Ursatz at
PR #97. Planned work, not status: see `Ursatz-Analyzer/Status.md` for what is built.

## Purpose change (record in `Reconciliation_Log.md` when the first branch merges)

The Analyzer stops being a narrow key-and-chord demo. It becomes **the single analysis
pipeline** of the ecosystem:

1. A **library** (`libursatz_analyzer`) that the GUI and the Composer link, ending the GUI's
   hand-copied key estimation.
2. A **CLI** that writes two outputs per piece:
   - `<stem>.report.md`: a readable analysis written for a composer ("what is this piece doing,
     and how could I use it").
   - `<stem>.analysis.json`: a structured, versioned **analysis document** that the Composer
     and corpus statistics consume.
3. A **corpus mode** that analyzes a folder of pieces and writes style statistics.

It covers everything the library establishes, ordered by how much a composer gets from it:
harmony and cadences first, then phrase and form, then motives and melody, then rhythm,
texture, voice leading, schemas and counterpoint.

## How to read this

- Branch IDs (`A2.3`) are referenced from the Composer roadmap. Library prerequisites use the
  IDs in `Ursatz_Roadmap.md` (`L2.5`).
- Commit lines are title-only, `<type> - <Description>`.
- Each branch that needs new library work starts with a submodule bump to the Ursatz commit
  that delivers it.

## Phase overview

| Phase | Goal | Library prerequisites |
|---|---|---|
| A1 Foundation | Library target, structured output, current submodule | L1.1 |
| A2 Perception in the report | Meter, real chords, keys over time, outer voices | L2.1 to L2.8 |
| A3 Harmony for composers | Roman numerals, function, cadences, phrase model, harmonic vocabulary | L3.1 to L3.3 |
| A4 Phrase, form and motive | Ideas, sentences, periods, hybrids, motives and how they develop | L3.6, L3.7 |
| A5 Melody, rhythm and texture | Line shape, non-chord tones, meter, syncopation, texture | L3.5, L3.8, L3.9 |
| A6 Voice leading, counterpoint and schemas | Part-writing checks, schemas, sequences | L3.4, L3.10 |
| A7 Corpus and style | Many pieces in, style profile out | L5.3, L5.5 |
| A8 Consumers | GUI and Composer on the shared library | A1 to A7 as available |

---

## A1. Foundation

### A1.1 `chore/bump-ursatz-pr97`
- `chore - Bump Ursatz submodule to PR #97`
- `test - Update expected chords for subset-match chord identification`
- `docs - Note the submodule version in the README`

### A1.2 `refactor/analyzer-library-target`
Split the pipeline into a library the GUI and Composer can link.
- `build - Build the pipeline as the ursatz_analyzer static library`
- `refactor - Move the public API into include/ursatz_analyzer/`
- `refactor - Replace fixed 32-byte labels with owned strings`
- `refactor - Link the CLI against ursatz_analyzer`
- `test - Link existing tests against the library target`
- `docs - Document the library API in the README`

Done when: the CLI is a thin `main.c` over the library, and labels are no longer truncated.

### A1.3 `feat/analysis-document-json`
The structured output every later branch extends.
- `feat - Define the analysis document schema with a version field`
- `feat - Write piece metadata, key and chords to the analysis document`
- `feat - Add --json and --markdown output flags`
- `test - Validate the analysis document against the schema`
- `docs - Document the analysis document schema`

The schema has one section per analysis area (`meter`, `keys`, `chords`, `cadences`,
`phrases`, `form`, `motives`, `melody`, `rhythm`, `texture`, `voice_leading`, `schemas`). Every
claim carries its source framework, its time span, the note ids it covers, and its uncertainty,
mirroring the library's `Interpretation`.

### A1.4 `refactor/report-sections`
- `refactor - Render the Markdown report from the analysis document`
- `feat - Add a summary section at the top of the report`
- `test - Snapshot reports for the five reference pieces`

---

## A2. Perception in the report

### A2.1 `feat/report-meter-and-measures` (needs L2.1, L2.3)
- `chore - Bump Ursatz submodule for meter import`
- `feat - Report time signature, measure count and pickup`
- `feat - Locate every claim by measure and beat`
- `test - Measure numbers match the score for the reference pieces`

### A2.2 `feat/report-weighted-chords` (needs L2.4, L2.5)
- `chore - Bump Ursatz submodule for weighted chord identification`
- `refactor - Replace onset grouping with library sonority segmentation`
- `feat - Report each chord with its duration, inversion and runner-up`
- `feat - Mark non-chord tones by type`
- `test - Chord coverage of Field Nocturne No. 1 above 90%`

### A2.3 `feat/report-key-regions` (needs L1.4, L2.8)
- `chore - Bump Ursatz submodule for local key regions`
- `refactor - Replace whole-piece key estimation with key regions`
- `feat - Report tonicizations, modulations and pivot chords`
- `refactor - Remove the brute-force tonic scan`
- `test - Key regions for Clementi 36 and Mozart 545`

### A2.4 `feat/report-outer-voices` (needs L2.6)
- `chore - Bump Ursatz submodule for outer-voice lines`
- `feat - Report soprano and bass scale degrees per chord`
- `feat - Write soprano and bass lines to the analysis document`
- `test - Snapshot this report section for the reference pieces`

### A2.5 `feat/musicxml-input` (needs L2.9)
- `chore - Bump Ursatz submodule for MusicXML import`
- `feat - Accept MusicXML files as input`
- `test - Analyze the reference pieces from MusicXML and compare with MIDI`
- `docs - Document MusicXML input`

---

## A3. Harmony for composers

Highest value for writing music: what chords, in what order, doing what job, ending how.

### A3.1 `feat/report-roman-numerals` (needs L3.1)
- `feat - Report Roman numerals per chord in its local key`
- `feat - Report applied chords as V/x and viio/x`
- `feat - List the piece's chord vocabulary with counts`
- `test - Snapshot this report section for the reference pieces`

### A3.2 `feat/report-harmonic-function` (needs L3.2)
- `feat - Report harmonic function per chord`
- `feat - Report the phrase model (Tb, PD, D, Te) per phrase`
- `feat - Report six-four types and prolongations`
- `feat - Report strong and weak predominants`
- `test - Snapshot this report section for the reference pieces`

### A3.3 `feat/report-cadences` (needs L3.3)
- `feat - Report each cadence with type, measure and soprano degree`
- `feat - Report the cadence plan of the whole piece`
- `test - Snapshot this report section for the reference pieces`

### A3.4 `feat/report-progression-patterns`
What a composer reuses: the progressions this piece leans on.
- `feat - Find recurring progressions of two to six chords`
- `feat - Report the most common progressions and where they occur`
- `feat - Write progressions to the analysis document as reusable patterns`
- `test - Progression patterns in Bach 114`

### A3.5 `feat/report-harmonic-rhythm`
- `feat - Report how often the chord changes per measure`
- `feat - Report harmonic rhythm acceleration toward cadences`
- `test - Snapshot this report section for the reference pieces`

---

## A4. Phrase, form and motive

Second-highest value: how ideas are built and grown into phrases and sections. This is what
the Composer's bottom-up and top-down engines learn from.

### A4.1 `feat/report-phrases` (needs L3.6)
- `feat - Report phrases with length, start, end and cadence`
- `feat - Report ideas and subphrases inside each phrase`
- `feat - Label phrase beginnings, middles and endings`
- `test - Snapshot this report section for the reference pieces`

### A4.2 `feat/report-themes` (needs L3.6)
- `feat - Report sentences with presentation and continuation`
- `feat - Report periods with antecedent and consequent cadences`
- `feat - Report hybrid themes and compound periods`
- `feat - Report continuation traits (fragmentation, acceleration, sequence)`
- `test - Snapshot this report section for the reference pieces`

### A4.3 `feat/report-phrase-expansion` (needs L3.6, L1.5)
- `feat - Report expansions and contractions against the nominal phrase length`
- `test - Snapshot this report section for the reference pieces`

### A4.4 `feat/report-form-plan`
- `feat - Report the section plan with thematic returns`
- `feat - Write a form tree (form, section, phrase, subphrase, idea) to the analysis document`
- `test - Form tree for the reference pieces`

The form tree is the Composer's main input: the same shape it fills in when composing.

### A4.5 `feat/report-motives` (needs L3.7)
- `feat - Report motive families with every occurrence`
- `feat - Report how each occurrence was transformed (transposed, inverted, fragmented, augmented)`
- `feat - Link motives to the ideas and phrases that contain them`
- `test - Motive families in Mozart 545`

### A4.6 `feat/report-motivic-development`
How a motive is grown, the core skill the bottom-up composer imitates.
- `feat - Report the sequence of transformations applied to each motive through the piece`
- `feat - Write development chains to the analysis document`
- `test - Snapshot this report section for the reference pieces`

---

## A5. Melody, rhythm and texture

### A5.1 `feat/report-melody` (needs L3.5)
- `feat - Report range, climax and contour per phrase`
- `feat - Report leaps and their recovery`
- `feat - Report non-chord tones in the melody by type`
- `feat - Report scale-degree tendencies the melody follows or breaks`
- `test - Snapshot this report section for the reference pieces`

### A5.2 `feat/report-rhythm-meter` (needs L3.8)
- `feat - Report rhythmic motives and density per phrase`
- `feat - Report syncopation, hemiola and hypermeter`
- `feat - Report tuplets and borrowed divisions`
- `test - Snapshot this report section for the reference pieces`

### A5.3 `feat/report-texture` (needs L3.9)
- `feat - Report texture per passage and where it changes`
- `feat - Report accompaniment patterns (block, arpeggiated, Alberti, oom-pah)`
- `test - Snapshot this report section for the reference pieces`

### A5.4 `feat/report-markings` (needs L2.9)
- `feat - Report tempo, dynamics and articulation from notated input`
- `test - Snapshot this report section for the reference pieces`

---

## A6. Voice leading, counterpoint and schemas

### A6.1 `feat/report-voice-leading` (needs L2.7, L3.4)
- `feat - Report motion types between voices`
- `feat - Flag parallel fifths and octaves and unresolved tendency tones`
- `feat - Report doubling and spacing per chord`
- `test - Snapshot this report section for the reference pieces`

### A6.2 `feat/report-schemas-sequences` (needs L3.10)
- `feat - Report galant schemas with their stages`
- `feat - Report harmonic sequences with their kind and length`
- `test - Snapshot this report section for the reference pieces`

### A6.3 `feat/report-counterpoint-check`
For checking your own exercises and two-voice writing.
- `feat - Add a --check-counterpoint mode for two-voice input`
- `feat - Report species rule violations by bar`
- `test - Fux cantus firmus exercises`

---

## A7. Corpus and style

### A7.1 `feat/corpus-mode`
- `feat - Analyze every file in a folder`
- `feat - Write one analysis document per piece and a corpus index`
- `feat - Resume an interrupted corpus run`
- `test - Snapshot this report section for the reference pieces`

### A7.2 `feat/style-profile` (needs L5.3)
- `feat - Summarize a corpus into a style profile (progressions, cadences, forms, motives, rhythms)`
- `feat - Write the style profile as a StyleModel file the Composer reads`
- `feat - Compare two style profiles`
- `test - Style profile of the reference corpus`

### A7.3 `feat/similar-material` (needs L5.5)
- `feat - Find passages in a corpus similar to a given motive or progression`
- `test - Snapshot this report section for the reference pieces`

---

## A8. Consumers

### A8.1 GUI moves onto the library (tracked in Ursatz-GUI)
`refactor/use-analyzer-library` in Ursatz-GUI: link `ursatz_analyzer`, delete the copied key
estimation from `analysis_service.c`, render the analysis document's sections as tabs. Closes
the GUI duplication gap.

### A8.2 `feat/composer-api`
- `feat - Expose an API to analyze a generated fragment in memory`
- `feat - Return rule violations and style distance for a fragment`
- `docs - Document the API the Composer uses`
- `test - Snapshot this report section for the reference pieces`

This is how the Composer critiques its own candidates with the same analysis it learned from.
