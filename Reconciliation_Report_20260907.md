# Reconciliation Report — 2026-09-07

Compares the three repo self-analyses (Ursatz, Ursatz-Analyzer, Ursatz-GUI, all dated 2026-09-07) against System_Registry_v3.txt. Purpose: surface disagreements between what the code does and what project documentation claims, before any doc condensation work begins.

---

## 1. Registry status-field lag (Ursatz library)

The Registry's `Status` column has not been updated for several entities that the library self-analysis confirms are implemented with real logic, not stubs:

| Registry entry | Registry status | Self-analysis finding |
|---|---|---|
| Framework | Not Started | `theory/framework.h` implemented; `common-practice-minimal` is a real, built Framework target |
| Constraint | Not Started | `constraint_evaluate()` implemented with real null-checks; returns `CONSTRAINT_EVAL_NOT_IMPLEMENTED` as a documented placeholder — partial, not absent |
| Preference | Not Started | `preference_rank()` similarly a documented stub, not absent code |
| Key Estimation | Not Started | `key_estimation_analyze()` implemented, deterministic, fails explicitly rather than guessing |
| Chord Identification | Not Started | Implemented, exported as `extern const Analyzer` |
| MIDIImporter | Not Started | `midi_importer.c`, 349 lines, parses SMF 0/1, used in production by `ursatz-analyzer` |

By contrast, `EventRepository` and `InterpretationRepository` **were** correctly updated to `Implemented (minimal/personal-scale, feat/persistence-minimal, merged)`. The persistence-layer update appears to have been applied to the Registry; the analysis-layer (Key Estimation, Chord Identification, Framework, Constraint, Preference) and I/O-layer (MIDIImporter) updates were not.

**Action needed:** Registry status column requires a pass across L7/L8/L9 (theory, rules, analysis, io) rows specifically — the persistence rows are trustworthy, the rest are not.

---

## 2. Registry vs. repo docs — already resolved, repo doc stale

- **VoiceReference / RelationshipReference / ProvenanceReference**: The library self-analysis flags this as an open issue in `docs/ARCHITECTURE.md` ("proposed only, blocked" per an external Registry). This is **already resolved** — Registry v3's changelog explicitly closed this gate on 2026-09-04, marking all three `Implemented`. `docs/ARCHITECTURE.md` in the Ursatz repo was not updated to reflect this and should be corrected (repo-side fix, not a Registry action).

## 3. Registry vs. repo docs — consistent, no action needed

- **Theory → Corpus forward-dependency**: Library flags `Tendency`/`StatisticalModel`/`StyleModel` as speced to depend on a not-yet-existing Corpus concept. Registry confirms `Corpus`, `Tendency`, `StatisticalModel`, `StyleModel` are all `Not Started` (Phase 9/15). This is expected, not a discrepancy — the repo doc's framing as an "open issue" is accurate given current phase.

---

## 4. Cross-repo documentation gaps (not Registry issues, but relevant to condensation/deletion decisions)

- **`ursatz-analyzer` README**: title only, no build/usage/dependency info. Everything about usage was inferred from source by the self-analysis.
- **`ursatz-gui` README**: overstates current state — describes Corpus as a present-tense tool ("Analysis/Corpus now, Generator later") when no Corpus route or frontend behavior exists. Sidebar entry is decorative only.
- **`ursatz` README**: states L9 "has not started," but `io/` and `persistence/` (both L9) are partially implemented with real code. Predates that work.

These are repo-local README problems, not System Registry problems — worth fixing in each repo directly rather than in project-level docs, but noted here since they affect confidence in "what's actually built" claims generally.

---

## 5. Items neither confirmed nor contradicted (submodule not checked out)

Both `ursatz-analyzer` and `ursatz-gui` self-analyses note that `external/ursatz` was an unfetched/empty submodule in their working tree at analysis time. Their descriptions of Ursatz's own interface (MIDIImporter behavior, KeyEstimation/ChordIdentification input semantics, `common_practice_minimal_framework()`) are inferred from call sites in the consumer repos, not verified against Ursatz's actual source. The Ursatz self-analysis (run directly against the library) is the authoritative source for those interfaces if the three reports ever conflict.

---

## 6. Notable non-Registry findings worth carrying into doc/code follow-up (informational only)

- `ursatz-gui`: unseeded RNG (`generate_piece_id` uses `rand()` with no `srand()` call) — latent ID-collision risk across process restarts.
- `ursatz-gui`: frontend has zero code path to the fully-implemented Analyzer API (`app.js` never calls `/api/analyzer/pieces*`) — backend/frontend capability gap.
- `ursatz-analyzer`: silent truncation of chord/key labels at 32 bytes in three call sites, undocumented.
- `ursatz` library: `analysis/` header location is misleading — `chord_identification.c`/`key_estimation.c` live under `src/analysis/` but compile into `frameworks/common-practice-minimal`, not `libursatz` itself. Anyone linking only `libursatz` won't find those symbols.

---

## Summary

The single actionable Registry-level finding is **§1**: six entities (Framework, Constraint, Preference, Key Estimation, Chord Identification, MIDIImporter) show `Not Started` in the Registry despite confirmed real implementation. This should be corrected in the Registry before any dependent documentation is condensed or rewritten, since several other project docs likely inherit their "not started" framing from these same rows.

Everything else is either already consistent, already resolved, or a repo-local README issue outside the Registry's scope.
