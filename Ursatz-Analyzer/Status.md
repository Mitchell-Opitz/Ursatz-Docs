# Ursatz-Analyzer — Status

**Last verified against repo state:** 2026-09-13, commit `38b72a9` (PR #4). See
`Reconciliation_Log.md` for how this was checked.

## Purpose

A small C11 command-line tool that turns a MIDI file into a human-readable Markdown analysis
report: estimates the piece's global key and, for each group of simultaneously-onset notes,
identifies the most likely chord. A thin consumer/demonstration app on top of the `Ursatz`
library (`external/ursatz`, git submodule) — supplies no music-theory logic of its own, only
the judgment calls Ursatz itself leaves open (how to group notes into sonorities, how to
search candidate tonics, how to map scale degrees) and the Markdown rendering.

## Current implementation

**End-to-end pipeline, working:** file read → import → onset-based sonority grouping →
global key search (all 12 candidate tonics, major/natural-minor only) → per-group chord
identification (scale-degree-based, non-diatonic notes dropped rather than failing the whole
group) → Markdown rendering → file write.

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

CLI: `ursatz-analyzer <midi-file-path>` — writes `<stem>.report.md` next to the input;
exits 1 with a usage/error message on bad input, allocation failure, or write failure.

## Current state relative to the ecosystem

**This repo is genuinely low-churn and does not track the Ursatz library's Framework Stage
work.** Only 4 PRs exist total; the most recent (#4) is a narrow submodule bump for a
key-tolerance fix, not new functionality. It still only does key + chord identification — no
Cadence/Motif/Period/Sentence/Form-level analysis exists here (that wiring only happened in
Ursatz-GUI). This is not staleness to fix; it reflects the repo's actual, narrower scope as a
demonstration CLI, distinct from Ursatz-GUI's broader consumer role. If this repo's purpose
is meant to expand (e.g. to also exercise the Framework-stage analyzers), that's a purpose
change to record here and in `Overview/Master_Roadmap.md`, not a "catch-up" bug fix.

## Build & test

CMake ≥ 3.20, C11 compiler, CTest. `external/ursatz` submodule (pinned commit
`610e3c4...`) — no other third-party dependency. Two test executables
(`test_analysis_pipeline`, `test_report_output`) registered with CTest.

## What depends on this repo

Nothing — leaf/terminal CLI application.
