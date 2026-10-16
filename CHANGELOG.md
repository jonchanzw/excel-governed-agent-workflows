# Changelog

All notable changes to this framework are documented here. Rule IDs (`PROTO-n` / `RUN-n` / `PLAY-x`) are stable once published; structural changes are called out explicitly.

## [Unreleased]

- Added `examples/`: filled-in PLAY-0 run-start checklist and a complete worked reconciliation run (tracker rows, digest entries, mid-run systemic failure, handoff).
- README: prerequisites box, privacy note, limitations section, common-issues FAQ, MIT/version badges, headline reworded to "Get set up in one prompt", screenshot placeholders for post-first-run captures.

- Added `governed-excel-agent-setup.md` at repo root: single-file setup — setup prompt + full guide embedded. End users attach one file to Copilot agent mode and the agent builds the full 11-tab workbook.
- README rewritten to lead with the one-file setup flow.

## [merged-1.2] — 2026-10-16

- Initial public scaffold of the merged protocol + runtime + playbook guide (single-file edition).
- Added autonomy profiles A/B/C with escalation brakes invariant across profiles (`PROTO-3`).
- Added `RunControl` one-row run record and `RunID`/`PlaybookVersion` tagging in context packs (`RUN-3`, `RUN-8`).
- Added workbook protection rules (`RUN-5 P11`), external-data authorisation path (`RUN-5 P10`), and single-session concurrency rule (`RUN-6`).
- Added adaptive sizing for all list lengths, with mandatory truncation records (`RUN-5 P8`).
- Base skill `excel-single-agent-context` included under `skill/`.
