# Master Roadmap — Ursatz Ecosystem

**Last verified against repo state:** 2026-10-05 (Ursatz @ PR #97, Ursatz-GUI @ PR #27,
Ursatz-Analyzer @ PR #5). See `Reconciliation_Log.md` for how this was checked.

## What this project is

**Ursatz** is a general-purpose, theory-neutral computational music platform, written in C.
Facts about music (notes, timing, structure) are kept strictly separate from
*interpretations* of those facts (chord, key), which are only ever produced under an
explicit, named, swappable "Theory Framework," never assumed by the kernel.

**Long-term goal:** analyze real music, then generate original music, encoding real
music-theory knowledge as a rule/preference-based symbolic system, not ML. The owner's
personal goal behind it: a tool that helps them compose music of professional quality.

**Repos:**
- `Ursatz` — the library (theory-neutral kernel + pluggable frameworks). Detail:
  `Ursatz-Library/Status.md`.
- `Ursatz-Analyzer` — CLI consumer app, depends on Ursatz. Detail: `Ursatz-Analyzer/Status.md`.
- `Ursatz-GUI` — GUI consumer app. Detail: `Ursatz-GUI/Status.md`.
- `Ursatz-Composer` (planned, Phase 3) — generation engine consumer app. No repo yet.

## Cross-repo status summary

| Repo | Purpose | Current state | Depends on |
|---|---|---|---|
| Ursatz | Theory-neutral kernel + pluggable theory frameworks | L0 to L8 implemented, L9 partial. 17 frameworks, 16 of them rule libraries translated from Open Music Theory (PRs #49 to #97), not yet connected to analysis | — |
| Ursatz-Analyzer | CLI: MIDI → Markdown key/chord report | Working end-to-end, deliberately narrow (key + chord only). Submodule at Ursatz PR #47 | Ursatz |
| Ursatz-GUI | Browser UI: upload, analyze, browse, view score | Library/Collections/Analysis views live; 7 Framework-stage analyzers wired in; score rendering + claim highlighting. Submodule at Ursatz PR #47 | Ursatz |
| Ursatz-Composer | Generative composition engine | Not started | Ursatz (Phase 3 gate below) |

## Where the project actually stands (2026-10-05)

An analyzer for real music needs two halves:

1. **Perception:** imported notes → musical facts ("beat 1 is a V7 in first inversion, the
   soprano is on scale degree 2, it's a strong beat").
2. **Knowledge:** musical facts → meaning ("V7 to I in root position with do on top is a PAC;
   it ends the antecedent of a period").

The knowledge half is now large: tonality, harmony, counterpoint, voicing, meter, rhythm,
form and notation rules, all tested. The perception half is small: one framework
(`common-practice-minimal`), a fixed quarter-note sonority grid, chord matching that fails on
most real textures, and a MIDI importer that ignores meter. **Closing the perception gap is
the next phase of work.** No new rule libraries until it is.

## Process discipline (why this has worked — keep following it)

1. **Registry-first.** Nothing is implemented in Ursatz without first being recorded in the
   System Registry (`Ursatz-Library/Registry/`). The project owner is the checkpoint between
   architecture and implementation. *Lapsed once, PRs #49 to #97 (2026-10-02 to 10-04);
   back-filled 2026-10-05.*
2. **One scoped branch per task**, never a whole phase at once.
3. **Judgment calls get flagged before code is written**, and the architect reviews and
   approves first. This has caught real design issues early, including the Theory Framework
   circular-dependency split, keeping `constraint_evaluate` a kernel stub, the L5-vs-L6
   placement question, the phrase-segmentation naming collision, and the Relationship-chain
   mechanism for Period/Sentence/Form.
4. **Root-cause fixes over symptom patches.** For example, the key-detection exact-match bug
   was fixed by generalizing existing chord-level tolerance to key-level, not patched around.
5. **Disclosed scope cuts, not silent gaps.** Every "not built yet" is documented at the
   point of the gap; see each repo's `Known_Gaps.md`.
6. **Investigate before implementing, AND before deciding.** A mid-session architecture
   decision on 2026-09-09 (relocating Motif/Cadence/Theme/PhraseOccurrence from L5 to L6) was
   made without checking real repo state, was wrong, and had to be reverted; see
   `Reconciliation_Log.md`.
7. **Documentation is verified, not assumed.** Treat any status in this repo as unverified
   until a dated log entry or a `Status.md` header confirms when it was last checked.
8. **Perception path rule (new, 2026-10-05).** Don't start a new framework or rule until the
   analysis layer can produce its inputs from imported music, or the same branch builds that
   path. Knowledge without a way to reach it is shelf-ware.
9. **Calibration reminder:** process quality and percent-of-vision-complete are different
   numbers. Overall completion is still a small fraction of the long-term vision.

## Phase overview

| Phase | Repo | Status |
|---|---|---|
| 0 | Ursatz | Done |
| 1 | Ursatz-Analyzer | Done |
| 2 | Ursatz + Ursatz-GUI | Done |
| 2.5 | Ursatz (Framework Stage: analyzers, seventh chords, Relationship use) | Code complete. **Corpus verification not complete**: the 5-piece reference set has had only Bach checked by hand. Blocked in practice on perception quality: chord ID resolves only 61/394 beats of Field 1 |
| 2.6 | Ursatz (framework rule libraries, PRs #49 to #97) | Done as rule libraries; not connected. Built ahead of plan and outside Registry-first; reconciled 2026-10-05 |
| **2.7** | **Ursatz, then both apps (perception: MIDI → labeled musical facts)** | **Next.** See below |
| 3 | Ursatz-Composer (new repo, not started) | Rule-checked sketching (Composer C1 to C3) may start once Ursatz L5.1, L5.4 and L5.6 merge; style learning and self-critique wait on 2.7 and the Analyzer. Plan: `Roadmaps/Ursatz-Composer_Roadmap.md` |

## Next: Phase 2.7, the perception slice

Goal: one end-to-end path from a MIDI file to framework-checked harmony and cadences, verified
on the 5-piece reference corpus in the GUI. In order, one scoped branch each:

1. **Housekeeping.** Run framework tests in CI; fix the `common-practice-minimal` guard
   allowlist. Decide `feat/phrase-expansion` (merge or drop).
2. **Meter from MIDI.** MIDIImporter reads time-signature meta-events and produces
   `TimeSignature`; measures and beat strength become real (via `standard-meter`).
3. **Chord identification that survives real texture.** Weight candidate chords by duration
   and beat strength, treating extra tones as `NonChordTone`s; adaptive sonority boundaries.
4. **Roman numerals and function.** Feed identified chords into `standard-tonality` (Roman
   numerals) and `standard-harmony` (function, cadences, phrase model), emitting
   Interpretations per the claim pattern.
5. **Soprano and bass lines.** Derive outer voices from the voice assignment, so PAC vs. IAC
   and bass-line rules get real inputs.
6. **Wire into the apps and verify.** Bump both submodules, surface the new claims in the GUI,
   and run all 5 corpus pieces. This closes Phase 2.5's exit criterion too.

Only after step 6 does new rule-library work resume, under rule 8 above.

Branch-and-commit plans for all three code repos: `Roadmaps/` (Ursatz, Ursatz-Analyzer,
Ursatz-Composer).

## Known ecosystem-wide blocker

Analysis quality on real MIDI is limited by perception, not by theory coverage: see Phase 2.7
and `Ursatz-Library/Known_Gaps.md` section 2.

## Governing principle for this document

Each phase is scoped to the minimum needed to unblock the next. Architecture detail
(Registry, layer specs) lives in `Ursatz-Library/`, not here. This document is cross-project
sequencing and current cross-repo state only. If a fact is specific to one repo, it belongs
in that repo's own `Status.md`, referenced from here, not duplicated here.
