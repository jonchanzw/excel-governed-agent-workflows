# Changelog

All notable changes to this framework are documented here. Rule IDs (`PROTO-n` / `RUN-n` / `PLAY-x`) are stable once published; structural changes are called out explicitly.

## [Unreleased]

## [merged-1.2] — 2026-10-16

- Initial public scaffold of the merged protocol + runtime + playbook guide (single-file edition).
- Added autonomy profiles A/B/C with escalation brakes invariant across profiles (`PROTO-3`).
- Added `RunControl` one-row run record and `RunID`/`PlaybookVersion` tagging in context packs (`RUN-3`, `RUN-8`).
- Added workbook protection rules (`RUN-5 P11`), external-data authorisation path (`RUN-5 P10`), and single-session concurrency rule (`RUN-6`).
- Added adaptive sizing for all list lengths, with mandatory truncation records (`RUN-5 P8`).
- Base skill `excel-single-agent-context` included under `skill/`.
