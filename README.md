# Electricity Plan Comparator

Ranks Energex-area retail electricity plans against your actual usage to find the cheapest option. Single static HTML page, no backend, no build step — runs entirely client-side.

**Live:** https://kmbrimble.github.io/electricity-plan-compare/

## How it works

1. Upload a usage CSV (hourly or daily kWh).
2. Upload an Energy Made Easy plans JSON export for your network area (Energex).
3. Optionally enter your current plan's rates for comparison.
4. It costs every plan against your usage and ranks them by estimated annual cost.

See the comments in `index.html` for the tariff-costing logic and known gotchas (rate units, controlled load handling, etc).

## Data

`data/` holds demo/default versions of the two input files — see `data/README.md`. You can also just upload your own files directly on the page; nothing is ever sent off your device.

## Running locally

No server needed — it's a static file. Just open `index.html` in a browser, or serve the repo root with any static file server if you prefer.

## Deployment

GitHub Pages, serving `main` from the repo root. Push to `main` and it deploys automatically.
