# Demo data

Drop your own files here — the app auto-loads them on page open (over http/https; falls back to manual upload if opened via `file://`).

- `usage-demo.csv` — your hourly (or daily) electricity usage export. See [Usage file format](#usage-file-format) below.
- `energex_plans_raw.json` — the Energy Made Easy plan data for the Energex network area, in the raw CDR `{plans: [...]}` shape (or a bare array).

Neither file is fabricated or edited by Claude — you provide them.

## Usage file format

The page reads **one CSV file** (JSON is not accepted; see [Converting your data](#converting-your-data)). Columns are found by header name, case-insensitively, in any order. Quoted fields are fine.

**Required**

| Column | Header must contain | Notes |
|---|---|---|
| Date or timestamp | `date` (a `datetime` or `timestamp` column is preferred when there is one) | See the date formats below. |
| Energy used | `kwh`, `usage` or `consumption` | kWh for that row. Not the solar column. |

If either is missing the page says `Could not find date/kWh columns in header: ...` and lists the headers it saw.

**Date formats.** `2026-08-01`, `2026-08-01 14:00`, `2026-08-01T14:00:00` (a trailing `Z` or `+10:00` is accepted and ignored), and `01/08/2026` (day/month/year) optionally followed by a time such as ` 9:00`. The hour is read **as printed**, with no timezone conversion, because that is how plan tariff windows (for example 16:00-20:59) are meant to be read.

**Hourly or daily.** If the date column carries a time, each row is one hour and time-of-use plans are costed exactly, hour by hour. If it carries only a date, rows are treated as daily totals and time-of-use plans use a typical load-shape estimate, flagged per row. A separate `hour` column is **not** read: put the time in the date column (or add a `datetime` column).

**Optional columns**

| Column | Header must contain | Used for |
|---|---|---|
| Solar export kWh | `solar`, `export` or `feed in` | Feed-in credit. If absent, solar credit shows as $0. |
| Solar credit $ | the solar words above plus `cost` or `credit` | Reconciliation against your retailer's own figures only. |
| Usage cost $ | `cost` or `charge` | Reconciliation against your retailer's own figures only. |
| Read quality | `quality` | `ACTUAL` rows count toward the hourly chart's "peak" records; any other value (for example `ESTIMATED`) still counts toward totals but not peaks. |

**What the status line tells you.** After loading, the page reports the number of days, the date range, and whether the data is hourly or daily. It also says when some days are partial (energy is then annualised over the days' worth of data actually present, supply charges over all calendar days) and how many rows were skipped. A row is skipped when its date or kWh is missing or unreadable.

## Converting your data

- **Different header names:** rename them to something the table above matches, for example `datetime,kwh`.
- **Wh instead of kWh:** divide by 1000 first; the page assumes kWh.
- **One column per hour (wide format):** not supported. Reshape to one row per hour.
- **Separate `date` and `hour` columns only:** combine them into one timestamp, for example `2026-08-01 14:00`.
- **JSON:** convert it to CSV first. For a list of `{"timestamp": ..., "kwh": ...}` objects:

```python
import json, csv
rows = json.load(open("usage.json"))
with open("usage.csv", "w", newline="") as f:
    w = csv.writer(f)
    w.writerow(["datetime", "kwh"])
    for r in rows:
        w.writerow([r["timestamp"], r["kwh"]])
```

Where your retailer's export is not in a shape the page reads, [open an issue](https://github.com/kmbrimble/electricity-plan-compare/issues) with the first few rows (remove anything personal) and a converter can be added for that source.
