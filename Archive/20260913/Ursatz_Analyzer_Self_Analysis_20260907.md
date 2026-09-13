# Ursatz-Analyzer — Repository Analysis

## 1. Purpose

Ursatz-Analyzer is a small C11 command-line tool that turns a MIDI file into a
human-readable Markdown analysis report: it estimates the piece's global key
and, for each group of simultaneously-onset notes, identifies the most likely
chord. It exists as a thin consumer/demonstration application on top of a
separate library, **Ursatz** (`external/ursatz`, pulled in as a git
submodule from `Mitchell-Opitz/Ursatz`), which supplies the actual music-
theory domain model and analyzers (MIDI import, entities/structure,
key-estimation, chord-identification). Ursatz-Analyzer's own code is
"glue": importing notes via Ursatz's MIDI importer, making a couple of
explicit judgment calls Ursatz itself leaves open (how to group notes into
sonorities, how to search candidate tonics, how to map scale degrees), and
rendering the results as a report file. The README is currently just the
repo title, so this purpose is inferred entirely from the source and its
comments, not from project documentation.

## 2. Public API surface

This is a CLI tool, not a service; there is one externally-invoked entry
point (the compiled binary) plus the internal library functions it and the
tests call. There are no network endpoints.

### CLI

```
ursatz-analyzer <midi-file-path>
```
- Reads the given MIDI file, runs the analysis pipeline, and writes a
  Markdown report next to it (see `derive_report_path`, below).
- Exits 1 with a usage message if not given exactly one argument.
- Exits 1 with a `stderr` message on import failure, allocation failure,
  or a write failure; prints `Report written to <path>` and exits 0 on
  success.

### Library functions (`ursatz_analyzer_pipeline`, `src/pipeline/*.h`)

These are C functions, callable by any code linking the static library
(currently only `main.c` and the two test binaries):

- `MidiPipelineResult midi_pipeline_import_file(const char *path, ImportedNotes *out)` — reads a file from disk and imports it via Ursatz's `MIDIImporter`; populates `*out` on success, otherwise prints an error to `stderr`.
- `void imported_notes_destroy(ImportedNotes *notes)` — frees an `ImportedNotes` and everything it owns.
- `bool segment_by_onset(NoteEvent *const *events, size_t count, SonorityGrouping *out)` — groups time-ordered notes into `SonorityGroup`s by exact onset-time equality.
- `void sonority_grouping_destroy(SonorityGrouping *grouping)` — frees a `SonorityGrouping`.
- `int pitch_class_of(long midi_note_number)` — reduces a MIDI note number to a 0–11 pitch class.
- `PITCH_CLASS_NAMES[12]` — pitch-class-index-to-name lookup table (`"C"`…`"B"`).
- `KeyEstimate estimate_global_key(NoteEvent *const *events, PitchNumericValue *const *pitch_values, size_t count)` — searches all 12 candidate tonics and returns the first major/natural-minor key Ursatz's `KeyEstimation` recognizes for the whole note sequence.
- `ChordEstimate identify_chord_for_group(const SonorityGroup *group, NoteEvent *const *events, PitchNumericValue *const *pitch_values, const KeyEstimate *key)` — computes scale degrees for one onset group under the given key and asks Ursatz's `ChordIdentification` for a chord label.
- `bool run_analysis_pipeline(const ImportedNotes *notes, AnalysisReport *report)` — orchestrates the above: one global key estimate, one chord identification per sonority group, collected into `report`.
- `void analysis_report_destroy(AnalysisReport *report)` — frees an `AnalysisReport`.
- `char *markdown_report_render(const char *source_filename, const AnalysisReport *report)` — renders a report to a newly-allocated Markdown string (caller `free()`s it).
- `char *derive_report_path(const char *input_path)` — derives `<stem>.report.md` from an input path (e.g. `song.mid` → `song.report.md`).
- `bool write_report_file(const char *path, const char *markdown)` — writes a string to a file, overwriting any existing content.

## 3. Internal architecture

```
main.c
  └─ midi_pipeline (io/MIDIImporter wrapper)  →  ImportedNotes
       └─ analysis_pipeline (orchestrator)
            ├─ segmentation        (groups notes into onset-simultaneous SonorityGroups)
            ├─ key_pipeline        (builds a whole-piece Structure, tries 12 tonics against Ursatz's KeyEstimation)
            └─ chord_pipeline      (per group: pitch class → scale degree → Ursatz's ChordIdentification)
                 └─ pitch_class_util (shared MIDI-note-number → pitch-class helper)
       └─ analysis_report      (plain data type: array of {time, chord label, key label})
       └─ markdown_report      (renders AnalysisReport → Markdown string)
       └─ report_writer        (derives output path, writes file)
```

- `analysis_pipeline.c` is the only module that calls both `key_pipeline` and
  `chord_pipeline`, and is the sole place their outputs are joined into one
  `AnalysisReport` (one key estimate for the whole file, reused per entry).
- `key_pipeline` and `chord_pipeline` both independently construct Ursatz
  domain objects (`Structure`, `Sonority`, `EntityID`s) and both do their own
  pitch-class → scale-degree math to feed Ursatz's `KeyEstimation` /
  `ChordIdentification` analyzers — there's no shared "build entities for
  Ursatz" helper between them.
- `pitch_class_util` is the one piece of code shared by both.
- `segmentation` is pure grouping logic with no dependency on Ursatz's
  analyzers, only on its `NoteEvent`/time types.
- `markdown_report` and `report_writer` are purely about output; they know
  nothing about MIDI, keys, or chords — they only consume the plain
  `AnalysisReport` struct.
- `main.c` wires the four stages together and is the only place error
  messages for "usage" and "unexpected top-level failure" are printed
  (per-stage functions print their own `stderr` diagnostics for expected
  failures).

## 4. Dependencies

**Depends on:**
- **Ursatz** (`Mitchell-Opitz/Ursatz`, `external/ursatz` git submodule) — the
  only external dependency. Supplies:
  - `io/midi_importer.h` (MIDI parsing → `NoteEvent`/`PitchNumericValue`)
  - `analysis/key_estimation.h`, `analysis/chord_identification.h` (the two analyzers)
  - `core/*`, `identity/*`, `structure/structure.h`, `semantics/sonority.h`, `time/*` (the domain/entity model Ursatz-Analyzer must construct objects in, to call the analyzers)
  - a `common_practice_minimal` CMake target providing `common_practice_minimal_framework()` (the music-theory "Framework" passed to both analyzers)
  - `ursatz` and `common_practice_minimal` CMake link targets, both added via `add_subdirectory(external/ursatz)` in the top-level `CMakeLists.txt`.
  - **Note:** in this checkout, `external/ursatz` is present only as an
    unfetched submodule pointer (the working tree is empty) — the CI
    workflow (`.github/workflows/ci.yml`) fetches it using a
    `URSATZ_SUBMODULE_TOKEN` secret, implying the Ursatz repo is private.
    I could not inspect Ursatz's actual headers/behavior directly; the
    interface described above is inferred from Ursatz-Analyzer's `#include`s
    and call sites, not verified against the Ursatz source.
- **CMake ≥ 3.20**, a C11 compiler, and CTest (via `enable_testing()`) — build/test tooling, no other third-party libraries (no libc extensions beyond standard `stdio`/`stdlib`/`string`).
- **GitHub Actions** (`ubuntu-latest`) for CI: submodule checkout, configure, build, `ctest`.

**Depended on by:** Nothing else in this repository or visible elsewhere —
this is a leaf/terminal project (a CLI application), not a library other
code is shown importing. No evidence in the repo of other consumers.

## 5. Current implementation status

**Built and working (has code + a passing test path):**
- End-to-end MIDI → report pipeline: file read → import → onset segmentation
  → global key search → per-group chord identification → Markdown rendering
  → file write.
- Onset-based sonority grouping (`segment_by_onset`).
- Global key search over all 12 candidate tonics, major/minor only
  (`estimate_global_key`).
- Scale-degree-based chord identification per group, with non-diatonic notes
  dropped rather than failing the whole group (`identify_chord_for_group`).
- Output path derivation and file writing.
- CLI argument handling and error-path messaging for bad usage, unreadable
  files, malformed MIDI, and allocation failures.
- Two test executables (`test_analysis_pipeline`, `test_report_output`)
  registered with CTest; `test_analysis_pipeline` is an end-to-end test
  against a hand-built SMF file.

**Stubbed / planned / not present:**
- No `README.md` content beyond the title — no documented build/usage
  instructions exist in-repo (this document infers usage from `main.c` and
  CI).
- No handling of key estimation for anything other than major/natural minor,
  and no regional/modulating key analysis — the code explicitly treats the
  whole file as one region (documented as a deliberate judgment call, not a
  gap, but it is a real functional limitation).
- No merging of notes with overlapping-but-not-identical onsets (e.g. tied
  suspensions across a chord change) — documented limitation of
  `segment_by_onset`.
- No SMPTE-division MIDI support (delegated to/limited by Ursatz's
  importer, per the error enum `MIDI_IMPORT_ERR_UNSUPPORTED_DIVISION`).
- No installation/packaging, no versioned releases, no license file visible
  in the file listing.

## 6. Internal inconsistencies

- **README vs. reality**: `README.md` contains only the title `# Ursatz-Analyzer`,
  giving no indication of the CLI usage, build steps, or Ursatz submodule
  dependency that the code and CI actually require. Anyone relying on the
  README alone could not build or run this project.
- **Truncation risk in label fields, silently accepted twice over**: `AnalysisReportEntry.chord_label`/`key_label` and `KeyEstimate.label`/`ChordEstimate.label` are fixed `char[32]` buffers. `copy_label` (analysis_pipeline.c) and the direct `strncpy` calls in `key_pipeline.c`/`chord_pipeline.c` both silently truncate any claim label longer than 31 bytes coming back from Ursatz's analyzers, with no logging or truncation indicator. This is consistent behavior across the three sites (not a contradiction), but it is a silent-data-loss path with no comment acknowledging it, unlike the codebase's general practice of calling out judgment calls explicitly.
- **`analysis_report.c` is essentially dead weight as a translation unit**: it contains only `analysis_report_destroy`, a 6-line free function; `AnalysisReport`'s only other "logic" (allocation, population) lives in `analysis_pipeline.c`, not here, so the type and its lifecycle are split across two files for no structural reason — plausibly done to mirror the `.h`/`.c` pairing convention used elsewhere (`segmentation`, `midi_pipeline`, etc.) rather than because the type has enough behavior to warrant its own module.
- **Unused-argument coupling in `identify_chord_for_group`**: the function receives the full `events` array (not just the group's own notes) purely so `build_sonority` can look up `note_event_temporal_anchor`/`note_event_id` by index — this is fine, but it means every call site must keep passing the *entire* note array through, even though the function's actual working set is just `group`'s indices into it; nothing enforces that the indices in `group` are valid for the `events`/`pitch_values` arrays passed at any given call site (there's no bounds assertion), so a future caller passing mismatched groups/arrays would fail silently or read out of bounds — not a bug today (there's only one call site, in `analysis_pipeline.c`, which passes matching data), but a latent assumption that isn't checked.
- **CI's private-submodule assumption is untestable from this checkout**: `ci.yml` assumes `URSATZ_SUBMODULE_TOKEN` gives access to a private `Mitchell-Opitz/Ursatz` repo; in this analysis session, `external/ursatz` is an empty, unfetched submodule directory, so none of the claims above about Ursatz's actual interface (e.g. exact `KeyEstimationInput`/`ChordIdentificationInput` field semantics, what `common_practice_minimal_framework()` returns) could be verified against Ursatz's source — they are inferred solely from how Ursatz-Analyzer calls them.
- **Time-convention coupling documented as fragile, and never re-checked**: `markdown_report.h`'s doc comment explicitly says the `beats = 2 × numerator / denominator` conversion "relies on the current single NoteEvent producer's fixed convention" and would need revisiting for a different importer — this is a self-acknowledged, not-yet-triggered fragility (a TODO-in-disguise) rather than a live inconsistency, since there is currently only one producer (`midi_importer`), but nothing in the code enforces or asserts the assumed denominator convention if that changed.
