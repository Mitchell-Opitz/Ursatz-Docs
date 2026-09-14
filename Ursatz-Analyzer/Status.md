# Ursatz-Analyzer — Status

**Last verified against repo state:** 2026-09-14, commit `<pending>` (PR #5, key-detection
fixes). See `Reconciliation_Log.md` for how this was checked.

## Purpose

A small C11 command-line tool that turns a MIDI file into a human-readable Markdown analysis
report. It estimates the piece's global key, and for each group of simultaneously-onset notes,
identifies the most likely chord. It's a thin consumer/demonstration app on top of the `Ursatz`
library (`external/ursatz`, git submodule), supplying no music-theory logic of its own, only
the judgment calls Ursatz itself leaves open (how to group notes into sonorities, how to
search candidate tonics, how to map scale degrees) and the Markdown rendering.

## Current implementation

**End-to-end pipeline, working.** File read, then import, then onset-based sonority grouping,
then global key resolution, then per-group chord identification (scale-degree-based,
non-diatonic notes dropped rather than failing the whole group), then Markdown rendering, then
file write.

**Key resolution, as of PR #5, tries three sources in order:** (1) Ursatz's MIDI key-signature
meta-event scanner (PR #47), taken as authoritative with silent precedence over the other two
when present (no surfaced conflict warning, even for a modulating piece where the meta-event
and the chord-derived tonic disagree); (2) a chord-based fallback that tries the **closing**
chord's root before the opening chord's (opening isn't reliably tonic — can be an anacrusis or
dominant harmony; fixed mid-branch after the regression corpus caught the old opening-first
order); (3) the brute-force 12-candidate-tonic search (`key_identification.c`), unchanged and
still carrying the known one-directional `matches_collection` containment bug (see
`Ursatz-Library/Known_Gaps.md`), now rarely reached. Flat-spelling (e.g. "Eb major" instead of
"D# major") is applied via minimal-accidental spelling (circle-of-fifths sharp/flat-count
comparison per tonic) to the fallback-derived key label only; it does not touch chord/note
spelling elsewhere in the report or in Ursatz's own `NoteEvent` (sharps-only by contract).

```
main.c
  └─ midi_pipeline (io/MIDIImporter wrapper) → ImportedNotes
       └─ analysis_pipeline (orchestrator)
            ├─ segmentation        (exact-onset-equality sonority grouping)
            ├─ key_pipeline        (whole-piece Structure, tries 12 tonics)
            └─ chord_pipeline      (per group: pitch class → scale degree → chord)
                 └─ pitch_class_util (shared MIDI-note-number → pitch-class helper)
       └─ analysis_report / markdown_report / report_writer (output only)
```

CLI: `ursatz-analyzer <midi-file-path>`, which writes `<stem>.report.md` next to the input,
and exits 1 with a usage/error message on bad input, allocation failure, or write failure.

## Current state relative to the ecosystem

**This repo is genuinely low-churn and does not track the Ursatz library's Framework Stage
work.** Only 4 PRs exist total; the most recent (#4) is a narrow submodule bump for a
key-tolerance fix, not new functionality. It still only does key + chord identification, with
no Cadence/Motif/Period/Sentence/Form-level analysis existing here (that wiring only happened in
Ursatz-GUI). This is not staleness to fix, but rather a reflection of the repo's actual, narrower
scope as a demonstration CLI, distinct from Ursatz-GUI's broader consumer role. If this repo's
purpose is meant to expand (e.g. to also exercise the Framework-stage analyzers), that's a
purpose change to record here and in `Overview/Master_Roadmap.md`, not a "catch-up" bug fix.

## Build & test

CMake ≥ 3.20, C11 compiler, CTest. `external/ursatz` submodule (pinned commit
`610e3c4...`), with no other third-party dependency. Two test executables
(`test_analysis_pipeline`, `test_report_output`) are registered with CTest.

## What depends on this repo

Nothing. This is a leaf/terminal CLI application.
