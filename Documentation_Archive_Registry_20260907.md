# Documentation Archive Registry

Full inventory of everything currently in local Archive (14 files moved out of Project Files on 2026-09-07, three of which were fully deleted from Project Files that same day). This is the reference for what exists and what it contains, so any file can be requested back by name without guessing.

Last updated: 2026-09-07 (post-condensing pass)

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
