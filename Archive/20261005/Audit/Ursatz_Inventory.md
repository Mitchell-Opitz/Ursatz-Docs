# Ursatz Inventory: PRs #49 to #97

**Audited:** 2026-10-05, Ursatz `main` at `090695c` (PR #97). Baseline is PR #48 (`310ea46`),
the last state this docs repo describes. All 49 PRs merged 2026-10-02 to 2026-10-04.

Totals: 997 files changed, ~93,460 lines added, 11 removed. About 82,100 lines are in 16 new
frameworks (about 36,000 of them tests). About 4,700 lines are new kernel code in `src/`.
About 3,300 lines are new kernel tests.

## Build and test results (run during this audit)

| Check | Result |
|---|---|
| Top-level build (`cmake -S . -B build`) | Clean, 0 warnings |
| Top-level `ctest` (what CI runs) | 220/220 pass, including valgrind and architecture-guard tests |
| All framework test suites (built via a wrapper, since CI never builds them) | 247/248 pass |
| The one failure | `architecture_common_practice_minimal`: its allowlist doesn't include its own `chord_vocabulary_seventh.h` / `seventh_identification.h`. Predates PR #49 (from the 2026-09-09 seventh-chord work). Never caught because it never runs in CI. |

## New frameworks (`frameworks/`)

Each one is a Framework extension module outside `src/theory/`, the same shape as
`common-practice-minimal`. Each has a `README.md`, a lazily built `Framework` singleton,
`Rule`/`Constraint` tokens per rule, `*_rule_holds()` predicates, hard/soft violation bitmasks,
and its own architecture-guard test. Inputs are caller-supplied plain data (scale degrees,
spellings, flags), not NoteEvents.

| Framework | PRs | Lines (.c/.h) | Test files | Depends on | Scope (from its README) |
|---|---|---|---|---|---|
| standard-notation | #49-51, touched by #53, #54, #57, #58 | 3,466 | 16 | ursatz | Staff, clefs (incl. octave-shifted, neutral), ledger lines, stems, accidentals, half/whole steps |
| standard-markings | #52 | 2,341 | 7 | ursatz | Dynamics order, articulation classes, tempo ordering, repeat/D.C./D.S. playback order |
| western-periods | #52 | 579 | 3 | ursatz | Six stylistic periods with overlapping year ranges |
| standard-rhythm | #53, #56 | 1,964 | 9 | ursatz | Note values (US/UK names), dots, ties, rests, tuplets |
| standard-meter | #54-56 | 3,759 | 14 | ursatz | Simple/compound meter, beats, beaming, anacrusis, counting, hypermeter, syncopation |
| standard-tonality | #57-63 | 15,882 | 33 | ursatz | Major/minor/modal/pentatonic/chromatic scales, solfege, key signatures, circle of fifths, intervals, triads, sevenths, chord symbols, Roman numerals |
| standard-texture | #64 | 1,459 | 5 | ursatz | Monophony/heterophony/homophony/polyphony classification |
| standard-voicing | #65 | 2,217 | 9 | ursatz | Strict SATB chord voicing rules (range, spacing, crossing, doubling) |
| standard-counterpoint | #66-73 | 18,795 | 48 | ursatz | Fux species 1 to 5, cantus firmus rules, Huron tendencies |
| sixteenth-century-style | #74 | 3,740 | 9 | **standard-counterpoint**, ursatz | Palestrina-style imitative polyphony |
| high-baroque-fugue | #75 | 2,661 | 7 | **standard-counterpoint**, ursatz | Fugal exposition (subject, answer, countersubject) |
| baroque-ground-bass | #76 | 2,243 | 7 | **standard-counterpoint**, ursatz | Ground bass, chaconne, passacaglia |
| galant-schemas | #77-78 | 2,522 | 8 | **standard-counterpoint**, ursatz | Gjerdingen schemas (prototype vs. exemplar) |
| scale-harmonization | #79 | 1,832 | 6 | **galant-schemas**, **standard-counterpoint**, ursatz | Rule of the octave, sequences over a bass scale |
| standard-harmony | #80-94 | 15,463 | 46 | ursatz | Harmonic function, phrase model, cadences, V7 resolution, predominants, embellishing/six-four chords, prolongation, plagal, mediant, tonicization, modulation |
| standard-form | #95-97 | 3,191 | 14 | ursatz | Form levels, motives, ideas, sentence, period, compound period, hybrid forms |

Nine READMEs cite Open Music Theory as the source.

## New kernel types (`src/`)

"Consumers" means non-test code that includes the header. **No new kernel type is used by
any other kernel code**, apart from a few `src/` cross-includes inside `pitch/` and `time/`.

### L2 Time / Pitch

| Type | PR | Consumers |
|---|---|---|
| `pitch/Clef`, `pitch/ClefResolution`, `pitch/StaffPosition` | #49, #50 | standard-notation |
| `pitch/Accidental` | #51 | standard-notation |
| `pitch/KeySignature` | #57 | standard-notation, standard-tonality |
| `pitch/GenericInterval` | #60 | standard-tonality |
| `time/MetronomeMarking` | #52 | standard-markings |
| `time/HistoricalPeriod` | #52 | western-periods |
| `time/NoteValue` | #53 | standard-rhythm, standard-meter |
| `time/TimeSignature` | #54 | standard-meter |
| `time/Tuplet` | #56 | standard-rhythm, standard-meter |

### L3 Events

| Type | PR | Consumers |
|---|---|---|
| `events/TempoMarking` | #52 | none in code (named in a standard-markings comment) |
| `events/StructuralMarker` | #52 | standard-markings |

### L4 Containers

| Type | PR | Consumers |
|---|---|---|
| `containers/Idea`, `containers/Subphrase` | #95, #96 | standard-form |

### L6 Interpretation

| Type | PR | Consumers |
|---|---|---|
| `interpretation/Texture` | #64 | standard-texture |

### L6 Semantics: 32 plain enums

Each header says it "carries no EntityID ... and no framework claim" and that its meaning is a
framework's convention.

| Enum | PR | Consumers |
|---|---|---|
| VoicePart | #65 | standard-voicing, standard-harmony, high-baroque-fugue |
| MotionType | #66 | standard-counterpoint |
| CounterpointSpecies | #67 | standard-counterpoint |
| BeatStrength | #68 | standard-counterpoint, standard-harmony, galant-schemas |
| NonChordTone | #69-71, #83 | standard-counterpoint, standard-harmony |
| ImitationAnswer | #74 | sixteenth-century-style, high-baroque-fugue |
| FugueRole | #75 | high-baroque-fugue |
| SubjectPart, SubjectCharacter | #75 | **none** (tests only) |
| OstinatoType, ImitativeRole | #76 | baroque-ground-bass |
| SchemaFunction | #77 | galant-schemas |
| SchemaSense | #78 | **none** (tests only; named in comments) |
| ScaleDirection, SequenceKind | #79 | scale-harmonization |
| HarmonicFunction, CadenceType | #80 | standard-harmony (CadenceType also standard-form) |
| SeventhResolution | #81 | standard-harmony |
| PhraseModelLabel | #86 | standard-harmony |
| SixFourType | #88 | standard-harmony |
| AppliedChordKind | #93 | standard-harmony |
| ModulationKind, KeyChangeExtent | #94 | standard-harmony |
| FormLevel, MotiveKind, PhraseRole, PhraseBeginning, PhraseEnding, IdeaLabel, SubphraseRole | #95, #96 | standard-form |
| ContinuationTrait, PhraseLevelForm | #96, #97 | standard-form |

### L8 Transform

| Type | PR | Consumers |
|---|---|---|
| `transform/IntervallicManipulation` (a 12th concrete transform) | #95 | standard-form |

## In-repo docs (Ursatz repo itself)

Updated alongside the code: `docs/modules/{events,interpretation,pitch,semantics,theory,time}.md`
plus each framework's `README.md`. The in-repo docs are current. This docs repo is what fell
behind.

## Not merged

`origin/feat/phrase-expansion`: 14 commits ahead of `main` (standard-form phrase expansion and
contraction). No PR merged.
