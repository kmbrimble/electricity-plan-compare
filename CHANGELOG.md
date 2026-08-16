# Changelog

All notable changes to this project are documented here. Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- Usage CSV and plans JSON now auto-load from `data/` on page open (GitHub Pages / any http server; falls back to manual upload when opened via `file://`).
- Dark-mode-only UI redesign; results table filters (retailer, tariff type, controlled load, sort); input cards collapse into a summary after running a comparison.
- "Hourly usage" tab: a per-hour-of-day bar chart (avg/min/max) for any month in the uploaded usage data, with month navigation, peak-hourly/peak-daily/weekday-vs-weekend stat callouts, and a single-day drill-down view. Requires interval (hourly) usage data.
- Solar export support: CSV parsing now optionally captures a solar export column (and Alinta's own per-interval usage/solar cost columns, and read quality). Each plan's estimated annual cost is now netted against its solar feed-in tariff, shown as its own "Solar credit $/yr" column (and a diff column once you enter your current feed-in rate). A small reconciliation line compares Alinta's own charged/credited totals for the uploaded period against the current-plan estimate, when those columns are present.
- Pay-on-time/direct-debit conditional discount toggle: opt-in checkbox in the Run comparison step that includes those percentOfBill discounts in the cost estimate (off by default).
- "Year 1 cost $" column: one-off sign-up credits are no longer silently dropped - they're shown as a separate one-time-only figure instead of being folded into the recurring "Est. annual cost" used for ranking.
- GitHub repo / changelog / author links in the footer.
- Alinta-specific caveat next to the controlled-load fields (its export combines main + CL usage, so only a daily-charge comparison is valid).
- Manual "Collapse back to summary" link so input cards can be re-collapsed after expanding, without needing to re-run the comparison.

### Fixed
- Every plan-side charge/rate is now grossed up by 10% for GST - the CDR feed reports them ex-GST, but real bills include it.
- Controlled-load daily charges are now summed across all of a plan's CL tariffs (e.g. CL1 + CL2) instead of only the first, both in the cost total and the (renamed) "CL daily charge" column.
- Tiered/block usage rates (e.g. a plan's first-N-kWh-per-day rate) are now costed against every tier, not just the first.
- Moved the Plan comparison / Hourly usage tab bar to the top of the page instead of below all 5 input steps.

## [0.1.0] - 2026-08-15

### Added
- Initial import: single-page electricity plan comparator (`index.html`), unchanged from its original form.
- Project scaffolding: README, data folder for demo files, GitHub Pages deployment.
