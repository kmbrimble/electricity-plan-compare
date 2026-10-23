# electricity-plan-compare

Ranks Energex-area retail electricity plans against a real usage export. `index.html` is the
entire app (~1750 lines): styles in a `<style>` block, logic in a `<script>` block. No backend,
no build step, no framework, no runtime dependencies. Everything runs client-side.

## Architecture
- Inputs: a usage CSV + an Energy Made Easy (CDR) plans JSON, parsed client-side — or
  `loadDefaults()` auto-fetches both from `data/` on page open (http(s) only, not `file://`).
- `parseUsageCsv` → `aggregateToDays` produces `usageDays`, the day-aggregated structure both
  the cost engine and the hourly chart read from. Extend it rather than re-parsing the CSV.
- Cost engine (`computeAnnualCostForPlan` and the functions under `--- Cost engine ---`) costs
  each plan day by day, then scales by `365 / days`. Solar feed-in is netted into `annualCost`
  but kept visible as its own field; sign-up credits are excluded from it and shown as
  `yearOneCost`.
- Two tabs (`switchTab`): the comparison table, and an hourly usage SVG chart needing only
  `usageDays`.
- `data/usage-demo.csv` and `data/energex_plans_raw.json` are supplied by the user and
  committed. Never fabricate or edit them — plausible fake data gives plausible wrong dollars.

## Test command
**There is no automated test harness.** No test files, no `npm test`, no CI. `playwright` is a
devDependency used only for ad-hoc render checks. The manual verification that replaces it,
from the repo root:

    python3 -m http.server 8791 &     # MUST be http:// — file:// cannot fetch data/
    node -e '
    const {chromium}=require("playwright");(async()=>{
      const b=await chromium.launch(),p=await b.newPage(),errs=[];
      p.on("console",m=>m.type()==="error"&&errs.push(m.text()));
      p.on("pageerror",e=>errs.push("pageerror: "+e.message));
      await p.goto("http://localhost:8791/index.html");
      await p.waitForTimeout(2500); await p.click("#runBtn"); await p.waitForTimeout(2000);
      console.log({rows: await p.$$eval("tbody tr",r=>r.length), errs}); await b.close();})()'

Green baseline with the committed demo data: **971 rows, `errs` empty** (992 plans − 21 Amber).
Anything less than 971 means plans stopped costing. Serving over `file://` is not a weaker
check, it is no check: `loadDefaults()` fails, no data loads, `#runBtn` stays disabled, and two
`URL scheme "file" is not supported` console errors appear that are expected, not regressions.

## Deploy and verify
GitHub Pages, legacy build from `main` at repo root, HTTPS enforced, no custom domain. Push to
`main` and it redeploys itself — there is no workflow file, no CI config and no Dockerfile in
the repo by design; Pages is configured in the GitHub repo settings, outside this repository.
Verify at https://kmbrimble.github.io/electricity-plan-compare/ — console clean, table
populated. Build status: `gh api repos/kmbrimble/electricity-plan-compare/pages`.

## Non-negotiable constraints
Each looks like a bug, poor practice, or a tidy-up. None is.

- **Single static `index.html`; no backend, build step, bundler or Docker.** Pages hosts it
  entirely; a build step buys nothing for one file and breaks the deploy path.
- **CDR rates and charges are in dollars, not cents** (`"0.4344"` = 43.44 c/kWh) — `toNum`,
  `toNumGst`, `extractSolarFeedInRate`. Verified against live cdr.energymadeeasy.gov.au data.
  Do not re-divide by 100.
- **`dailySupplyCharge` is singular** in the live schema. Not a typo.
- **`controlledLoad` is a sibling of `tariffPeriod`** on the contract, not nested inside a
  tariff period — `costControlledLoadDailyCharge`.
- **Only the CL *daily charge* is added, never CL energy re-costed at the CL rate** —
  `costControlledLoadDailyCharge`. The usage export already includes CL energy in the main
  totals; re-costing it double-counts.
- **Plan-side charges are grossed up 10% for GST** (`GST_MULTIPLIER`); the user's current-plan
  inputs and solar feed-in rates are not. The feed is ex-GST, bills are not, and the user's own
  figures come off a real bill.
- **Amber is excluded** (`EXCLUDED_RETAILERS`, 21 plans). Its filed rate is a nominal reference
  price, not what it bills.
- **Sign-up credits stay out of `annualCost`**, appearing only in `yearOneCost`
  (`extractSignUpCredit`). Folding a one-off into the recurring figure ranks a plan cheapest on
  a one-time bonus.
- **`solarCredit` is always a number** so its column never auto-hides (`applyColumnVisibility`);
  `solarCreditDiff` may be null and may hide. "$0 credit" is a real result.
- **CSV column-detection order in `parseUsageCsv` is deliberate** — solar-cost before solar and
  usage-cost, those before plain kWh, each excluding earlier matches' index. Reordering silently
  mis-assigns columns.
- **`isEstimated` rows count toward totals** but are excluded from "peak" records in the hourly
  chart.

## Code review
Context for `code-diff-reviewer`, `code-audit` and `code-security-audit`.

### Exposure
**Internet-facing, unauthenticated.** GitHub Pages at
https://kmbrimble.github.io/electricity-plan-compare/ — public repo, HTTPS enforced, served
straight from `main` at repo root. No Cloudflare tunnel, no reverse proxy, no auth layer, no
open ports, no server-side component. Everything committed is publicly readable, including
`data/usage-demo.csv` (a real household usage export) and the plan feed. At runtime the only
network calls are same-origin `fetch()` of `data/` plus outbound links to
energymadeeasy.gov.au; no user data leaves the browser.

### Modules that own data users rely on
Nothing is persisted — no storage of any kind, so a bug does not lose data. It produces a wrong
dollar figure driving a real retailer-switching decision. The money paths:

- `computeAnnualCostForPlan` → `costUsageAgainstTariffPeriod` → `costKwhAgainstTieredRates` /
  `costControlledLoadDailyCharge`, plus `toNumGst` — the ranking number itself.
- `parseUsageCsv` / `aggregateToDays` — silently `continue`s past unparseable rows. Dropped rows
  shorten the period and skew the `365 / days` annualisation of every plan at once.
- `extractSolarFeedInRate` / `computeSolarCreditForYear` / `extractSignUpCredit` — netted into,
  or split out of, that same ranked figure.
- The current-plan block in the `runBtn` handler (under `// Current plan comparison`) — the
  baseline every diff column is measured against, and the only place rates are entered in cents
  and divided by 100.

### Infrastructure this repo depends on but does not contain
**Nothing is injected, proxied or enforced from outside at runtime.** No secrets, no environment
variables, no template-supplied config, no sibling container, no reverse proxy adding auth, no
tunnel terminating TLS. Absent authentication, absent secret handling and absent server-side
validation are the design, not omissions — do not report them as defects. What genuinely lives
outside the repo:

- **GitHub Pages** — hosting, TLS, and the deploy itself. The missing workflow/CI/Dockerfile is
  deliberate (see Deploy and verify).
- **The CDR plan feed schema** (AER / Energy Made Easy). `data/energex_plans_raw.json` is a
  manual export; the schema is not versioned, validated or fixtured in-repo. The cost engine's
  field-shape assumptions are load-bearing and unenforced by anything here.
- **Playwright browser binaries**, in the container at `~/.cache/ms-playwright`. `node_modules/`
  is gitignored; `npm i` plus those binaries must exist before any render check.

### Test reality
Every claim below was checked against the committed `data/` files, not assumed.

Exercised by the render check above: single-rate and TOU costing, tiered/block rates (191 tariff
periods), controlled-load daily charges including multi-CL plans (235), guaranteed
`percentOfBill` discounts (48), guaranteed `percentOfUse` discounts (7, all CovaU Freedom; they
apply to usage charges only), sign-up credits (70), solar credit, the Alinta reconciliation
line, and `ESTIMATE` read-quality rows (8 of 6007).

**Never exercised by any test, manual or otherwise** — changed code here has nothing taking the
branch, which in this codebase is the strongest predictor of a real defect:

- The daily-totals-only load-shape fallback in `costUsageAgainstTariffPeriod`
  (`DEFAULT_TOU_WEIGHTS_BY_PREFIX`, `weightForBlockType`). The demo CSV is hourly, 24 rows/day,
  so the exact-match branch always wins.
- The unmatched-TOU-hour average fallback in the same function. All 581 TOU tariff periods in
  the feed cover all 24 hours on all 7 days, so `unmatchedKwh` is always 0.
- `selectTariffPeriodForDate` / `isMonthDayInRange` past the single-period early return — all
  992 plans have exactly one tariff period.
- The pay-on-time / direct-debit conditional discount branch. Off by default; 28 plans carry one.
- Both manual file-upload handlers — auto-load from `data/` bypasses them.
