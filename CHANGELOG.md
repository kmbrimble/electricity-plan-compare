# Changelog

All notable changes to this project are documented here. Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- Usage CSV and plans JSON now auto-load from `data/` on page open (GitHub Pages / any http server; falls back to manual upload when opened via `file://`).
- Dark-mode-only UI redesign; results table filters (retailer, tariff type, controlled load, sort); input cards collapse into a summary after running a comparison.
- "Hourly usage" tab: a per-hour-of-day bar chart (avg/min/max) for any month in the uploaded usage data, with month navigation, peak-hourly/peak-daily/weekday-vs-weekend stat callouts, and a single-day drill-down view. Requires interval (hourly) usage data.
- Solar export support: CSV parsing now optionally captures a solar export column (and Alinta's own per-interval usage/solar cost columns, and read quality). Each plan's estimated annual cost is now netted against its solar feed-in tariff, shown as its own "Solar credit $/yr" column (and a diff column once you enter your current feed-in rate). A small reconciliation line compares Alinta's own charged/credited totals for the uploaded period against the current-plan estimate, when those columns are present.

## [0.1.0] - 2026-08-15

### Added
- Initial import: single-page electricity plan comparator (`index.html`), unchanged from its original form.
- Project scaffolding: README, data folder for demo files, GitHub Pages deployment.
