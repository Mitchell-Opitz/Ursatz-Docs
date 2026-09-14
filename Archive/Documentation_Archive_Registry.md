# Documentation Archive Registry

Full inventory of everything in `Archive/`, across every archiving pass. This is the
reference for what exists and what it contains, so any file can be requested back by name
without guessing. Living index — update it whenever files move into `Archive/`; don't rely
on folder dates alone to explain what's inside them.

Last updated: 2026-09-13 (full documentation restructure — see `Reconciliation_Log.md`)

---

## 2026-09-13 — full restructure (`Archive/20260913/`)

All 14 root-level docs from the 2026-09-07 condensing pass were superseded by the current
`Overview/`, `Ursatz-Library/`, `Ursatz-Analyzer/`, `Ursatz-GUI/` structure (see the root
`README.md`). Moved to archive rather than deleted, since each still has historical value as
a point-in-time snapshot:

| Archived file | Superseded by |
|---|---|
| Master_Roadmap_v6_20260907.md | `Overview/Master_Roadmap.md` |
| Ursatz_Apps_Roadmap_v3_20260907.md | `Overview/Apps_Roadmap.md` |
| TDD_v5_20260907.md | `Overview/Architecture_Principles.md` + `Ursatz-Library/Technical_Design_Document.md` |
| Domain_Specification_v4_20260907.md | `Ursatz-Library/Domain_Specification.md` |
| API_Contract_Specification_v4_20260907.md | `Ursatz-Library/API_Contract_Specification.md` |
| Canonical_IR_Specification_v3_20260907.md | `Ursatz-Library/Canonical_IR_Specification.md` |
| Dependency_&_Layer_Specification_v5_20260907.md | `Ursatz-Library/Dependency_Layer_Specification.md` |
| Test_Specification_v3_20260907.md | `Ursatz-Library/Test_Specification.md` |
| System_Registry_v7_20260907.md | `Ursatz-Library/Registry/*.md` (split by layer) |
| Ursatz_Self_Analysis_20260907.md | `Ursatz-Library/Status.md` + `Known_Gaps.md` |
| Ursatz_Analyzer_Self_Analysis_20260907.md | `Ursatz-Analyzer/Status.md` + `Known_Gaps.md` |
| Ursatz_GUI_Self_Analysis_20260907.md | `Ursatz-GUI/Status.md` + `Known_Gaps.md` |
| Reconciliation_Report_20260907.md | `Reconciliation_Log.md` (2026-09-07 entry) |
| Documentation_Archive_Registry_20260907.md | this file (renamed, dropped date suffix — now a living index like everything else) |

**Why archived, not deleted:** each described real, dated project state accurately at the
time; none were wrong, just superseded by a structure better suited to fast lookup and easy
incremental updates as the ecosystem grows (see `Reconciliation_Log.md`'s 2026-09-13 entry
for the reasoning).

---

## Deleted from Project Files (archived only)

### Project_Update.txt
**What it was:** A point-in-time module-by-module review pass (L0 identity through early analysis-layer types), checking implemented code against a requirements list item by item.
**Why archived:** Superseded by later implementation and by the 2026-09-07 self-analysis reports + reconciliation report.

### cleanliness_audit.md
**What it was:** A read-only size/verbosity/maintainability audit of the Ursatz repo as of 2026-09-04.
**Why archived:** Point-in-time metrics snapshot, no longer current.

### reconciliation_report.md (original)
**What it was:** A read-only audit of the Ursatz repo at commit `8768d8e`, verifying `docs/ARCHITECTURE.md`'s claims layer by layer.
**Why archived:** Superseded by the 2026-09-07 Reconciliation_Report and more recent code state.

---

## Still in Project Files, pending final delete (not yet actioned)

### Ursatz_Full_Logic_Pass_v1.md
**What it is:** A layer-by-layer (L0→L9) planning document built from the original reconciliation_report.md's findings.
**Status:** Still marked for deletion — superseded by current self-analyses, source material now resolved. Not yet removed from Project Files.

### Ursatz_base_project_structure.txt
**What it is:** A raw Windows-path directory tree listing of the Ursatz repo — a filesystem snapshot, not authored documentation.
**Status:** Still marked for deletion — stale the moment the repo changes, trivially regenerable. Not yet removed from Project Files.

---

## Condensing pass — complete

The following were each superseded by a new version during the 2026-09-07 condensing pass. Old versions removed from Project Files; new versions added.

| Old file | New file | What changed |
|---|---|---|
| Ursatz_Apps_Roadmap_v1.md | **Ursatz_Apps_Roadmap_v2.md** | Corrected stale "PitchIdentity arithmetic gap" blocker (L8-Transform is done); updated Phase 1/2 status to current reality |
| Ursatz_Master_Roadmap_v2.md | **Master_Roadmap_v3.md** | Condensed (255→~106 lines); status section updated to reflect persistence merged, GUI backend done, frontend gap |
| Domain_Specification_v3.txt | **Domain_Specification_v4.md** | Condensed (896→~145 lines); resolved items (VoiceReference-family, PitchIdentity equivalence) marked resolved rather than proposed |
| API_Contract_Specification_v3.txt | **API_Contract_Specification_v4.md** | Condensed (435→~195 lines); same currency fixes as Domain Spec |
| Canonical_IR_Specification_v2.txt | **Canonical_IR_Specification_v3.md** | Condensed (253→~185 lines); reference-type authorization marked resolved |
| Test_specification_v2.txt | **Test_Specification_v3.md** | Condensed (483→~205 lines); VoiceReference-family tests marked runnable, not just written |
| Dependency___Layer_Specification_v2.txt | **Dependency_&_Layer_Specification_v3.md** | Condensed (233→~150 lines); both formerly-open issues (Interpretation→Framework, Theory→Corpus) marked resolved |
| TDD_v4.txt | **TDD_v5.md** | Condensed (5,830→~700 lines); Phase-1 object detail pointed to Domain/API/Canonical specs instead of re-derived; phase roadmap table updated with actual per-phase status |
| System_Registry_v3.txt | **System_Registry_v4.md** | Not condensed (already minimal per row) — Status column corrected for 32 rows with direct evidence (Framework, Constraint, Preference, Key Estimation, Chord Identification, MIDIImporter, Phase-1 kernel objects, PitchClass, FrameworkReference, all 11 Transform types). Broader status-lag across containers/structure/semantics/interpretation/theory/rules/generation/io/persistence modules flagged as known follow-up, not yet individually verified. |

**Also produced this pass:** Reconciliation_Report_20260907.md (compares the three self-analyses against the Registry) and this Documentation Archive Registry itself.

---

*(When Ursatz_Full_Logic_Pass_v1.md and Ursatz_base_project_structure.txt are actually deleted from Project Files, move their entries to the "Deleted" section above.)*
