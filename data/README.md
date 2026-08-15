# Demo data

Drop your own files here — the app will default to reading these (once that wiring is added; for now it's manual upload only via the page itself).

- `usage-demo.csv` — your hourly (or daily) electricity usage export. Needs a date/timestamp column and a kWh column; see `index.html`'s CSV parser for accepted formats.
- `energex_plans_raw.json` — the Energy Made Easy plan data for the Energex network area, in the raw CDR `{plans: [...]}` shape (or a bare array).

Neither file is fabricated or edited by Claude — you provide them.
