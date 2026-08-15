# Demo data

Drop your own files here — the app auto-loads them on page open (over http/https; falls back to manual upload if opened via `file://`).

- `usage-demo.csv` — your hourly (or daily) electricity usage export. Needs a date/timestamp column and a kWh column; see `index.html`'s CSV parser for accepted formats. Optional columns, auto-detected by header name: a solar export column (matched by `solar`/`export`/`feed in`), Alinta's own per-interval usage/solar cost columns, and a read-quality column (only "ACTUAL" rows count toward the hourly-chart "peak" stats; other values still count toward totals).
- `energex_plans_raw.json` — the Energy Made Easy plan data for the Energex network area, in the raw CDR `{plans: [...]}` shape (or a bare array).

Neither file is fabricated or edited by Claude — you provide them.
