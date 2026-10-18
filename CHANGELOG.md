# Changelog

## v1.0 — 8 Oct 2026

First public release of the anonymised sample.

### Added
- `FTL_Logistics_Expense_Tracker_SAMPLE.xlsx`: structured tables for own-fleet loads, market loads, monthly sale and KM run, with a `SUMIFS` monthly reconciliation and a year-filtered dashboard. The original sheets are kept (hidden) for traceability.
- Docs: [restructuring notes](docs/restructuring-notes.md), [data dictionary](docs/data-dictionary.md) and [formula reference](docs/formula-reference.md).
- README banner, badges, section links and screenshots.

### Fixed in the clean layer
Text dates in five shapes converted to real dates, hand-typed months derived from the date, provider and status labels standardised, 150 blank rows removed, truck numbers stored consistently as text, and the hand-typed Monthly Sale rebuilt from the tables (9 of 28 months had not tied to the load register). Column-by-column detail is in [`docs/restructuring-notes.md`](docs/restructuring-notes.md).

### Known issues
- The original `Daily Tracking` sheet still has 10 `#REF!` formulas. It is out of scope for this pass (see README, Validation).

## Repository maintenance

- 17 Oct 2026: added `.gitattributes` (workbook and images binary, LF for docs).
- 18 Oct 2026: added this changelog.
