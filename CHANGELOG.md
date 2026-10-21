# Changelog

All notable changes to this framework are documented here. Rule IDs (`PROTO-n` / `RUN-n` / `PLAY-x`) are stable once published; structural changes are called out explicitly.

## [merged-1.3] — 2026-10-19

- **Objective-driven setup:** the agent asks for the user's objective (plus data situation), then presents one combined proposal — restated objective, a drafted `DoneDefinition` goal contract (new `RunControl` field, RUN-3), and a tailored tab set — for a single confirmation before building.
- **RUN-1 core vs. conditional tabs:** `Tracker`, `ContextPack`, `Output`, `Digest`, `RunControl`, and rule tabs are always built; `Staging`, `Handoff`, `Cold` only when needed, with lazy creation when a run crosses a trigger.
- **Agent-drafted starting playbook:** setup now drafts PLAY-0 plus the first phase (minimal by instruction) for user approval — the heaviest manual step becomes a review.
- **Guided autonomy-profile choice** at setup end (plain-language A/B/C per PROTO-3), recorded in RunControl.
- **Progress visibility:** Digest entries open with a plain-language STATUS line including formula-computed % complete (RUN-8); the final Handoff doubles as the run's final report (goal met, exceptions, decisions — three lines, no new format).
- **Setup file navigation:** prompt and embedded guide now collapsed under expandable sections; the visible file is ~40 lines. Guide's legacy loading instructions slimmed; CONTRIBUTING now notes the two-location setup description and no-drift rule.

- Added `examples/`: filled-in PLAY-0 run-start checklist and a complete worked reconciliation run (tracker rows, digest entries, mid-run systemic failure, handoff).
- README: prerequisites box, privacy note, limitations section, common-issues FAQ, MIT/version badges, headline reworded to "Get set up in one prompt", screenshot placeholders for post-first-run captures.
- Fixed worked-example consistency bug: RunControl `CurrentPhase` now 2, matching tracker and checkpoint records.
- Removed `skill/SKILL.md` (superseded earlier edition of the framework). `governed-excel-agent-setup.md` is now the single drop-in file; rules of record live in `docs/governed-workflows-guide.md`. No parallel copies to drift.

- Added `governed-excel-agent-setup.md` at repo root: single-file setup — setup prompt + full guide embedded. End users attach one file to Copilot agent mode and the agent builds the full 11-tab workbook.
- README rewritten to lead with the one-file setup flow.

## [merged-1.2] — 2026-10-16

- Initial public scaffold of the merged protocol + runtime + playbook guide (single-file edition).
- Added autonomy profiles A/B/C with escalation brakes invariant across profiles (`PROTO-3`).
- Added `RunControl` one-row run record and `RunID`/`PlaybookVersion` tagging in context packs (`RUN-3`, `RUN-8`).
- Added workbook protection rules (`RUN-5 P11`), external-data authorisation path (`RUN-5 P10`), and single-session concurrency rule (`RUN-6`).
- Added adaptive sizing for all list lengths, with mandatory truncation records (`RUN-5 P8`).
- Base skill `excel-single-agent-context` included under `skill/`.
