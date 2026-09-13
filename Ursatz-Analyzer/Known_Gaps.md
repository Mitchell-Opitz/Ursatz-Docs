# Ursatz-Analyzer — Known Gaps

**Last verified against repo state:** 2026-09-13, commit `38b72a9` (PR #4). All items below
confirmed still present at this commit — none have changed since the prior pass.

- **README.md contains only the title** (`# Ursatz-Analyzer`) — no build/usage/dependency
  docs in-repo. Everything about usage is inferred from `main.c` and CI.
- **Silent label truncation.** `AnalysisReportEntry.chord_label`/`key_label`
  (`analysis_report.h`) and `KeyEstimate.label`/`ChordEstimate.label`
  (`chord_pipeline.h`/`key_pipeline.h`) are fixed `char[32]` buffers. `copy_label`
  (`analysis_pipeline.c`) and direct `strncpy` calls in `chord_pipeline.c`/`key_pipeline.c`
  all silently truncate any claim label longer than 31 bytes, with no logging or truncation
  indicator.
- **`analysis_report.c` is a 13-line file** containing only `#include`s and the
  `analysis_report_destroy` free function — the rest of `AnalysisReport`'s lifecycle lives in
  `analysis_pipeline.c`. Not a bug, just a thin translation unit mirroring the `.h`/`.c`
  convention used elsewhere.
- **No regional/modulating key analysis.** `key_pipeline.c`'s `build_whole_piece_region()`
  treats the entire file as one region; `is_major` is a binary major/not-major
  classification via `strstr(claim, "major")`. This is a deliberate judgment call
  (documented as such in the code), not an oversight, but it is a real functional
  limitation for any piece that modulates.
- **`segment_by_onset` requires exact onset-time equality** (`onset_equals` in
  `segmentation.c`) — no tolerance/overlap window, so overlapping-but-not-identical onsets
  (e.g. a tied suspension across a chord change) land in separate groups rather than being
  merged.
- **No SMPTE-division MIDI support** — delegated to/limited by Ursatz's importer
  (`MIDI_IMPORT_ERR_UNSUPPORTED_DIVISION`).
- **No installation/packaging, no versioned releases, no license file.**

None of these are staleness — this repo's own development has been quiet (one narrow
submodule bump since the last pass) and every item above was independently re-confirmed
against current source at the commit noted above.
