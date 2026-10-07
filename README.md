# Ops Reporting Dashboard

A one-page operations report for any chosen period, covering Sales, Project Delivery and People data from three different systems, with the numbers computed in code and the commentary written by Claude.

<!-- SCREENSHOT + DEMO VIDEO: added in the second pass -->

**Stack:** n8n · Claude · Supabase · HTML, CSS and JavaScript · **Live demo:** available on request

> **What's in this repo:** the dashboard (`index.html`). The n8n workflow that pulls the data, calculates the metrics and calls Claude isn't exported here yet. Its export is coming in a later commit. The workflow is described below so the dashboard makes sense, but its code can't be checked here until then.

## The problem

An operations lead needs one report across Sales, Delivery and People, but the data lives in three places: Google Sheets, Airtable and an internal API. Building that report by hand for every period is slow. Asking an AI to write it is risky, because a model can get the numbers wrong.

## Results

- **23 metrics on one page:** 8 for Sales, 7 for Project Delivery and 8 for People. Each is compared with the previous period of equal length.
- **4 breakdown charts:** revenue by lead source, delivery load by team, headcount by department and exits by department.
- **Any period** from 1 January 2025 to 30 June 2026, plus three ready-made ones (last 30 days, last 90 days, year to date).
- **AI commentary** (summary, operational risks and recommended actions) and **data-quality warnings** alongside the numbers.

## What it does

1. **Pick a period:** one of the three ready-made periods, or a custom start and end date.
2. **See the report.** If that period was calculated before, the stored report appears at once. If not, the page starts the workflow and shows its progress until the new report is stored. Tick **Recalculate even if stored** to rerun against the source systems.
3. **Read it:**
   - a tile per metric with its change, coloured by whether the change is good (for cost per lead, exits or delays, going down is good);
   - a chart per breakdown;
   - the AI analysis;
   - the data-quality section, which separates issues that affected this period from records flagged across the whole dataset.

## How it works

```
Dashboard (index.html)
  ├─ reads stored reports ─────────► Supabase: report_runs  (read-only public key)
  └─ new or recalculated range ────► n8n workflow (webhook)
                                       ├─ pulls Sales (Google Sheets), Delivery (Airtable), People (internal API)
                                       ├─ calculates the metrics deterministically
                                       ├─ sends a summary to Claude for the commentary
                                       └─ stores the report in Supabase
  dashboard polls report_runs every 3 s until the new report appears (up to 150 s)
```

| Part | Where | What it does |
| --- | --- | --- |
| Dashboard | `index.html` | One static page with no build step. It reads `report_runs` from Supabase, starts the workflow for new ranges and renders the report. It never calculates a metric itself. |
| Workflow | n8n (export coming) | Pulls the three sources, computes the metrics, asks Claude for the commentary and writes the report to Supabase. |
| Storage | Supabase | One `report_runs` row per period, holding the metrics, trends, data-quality results, AI insights and methodology notes. |

All periods are anchored to a reference date of **30 June 2026**, because the source data ends there.

## Design decisions

- **The numbers never come from the model.** Metrics are calculated in code. Claude only interprets a summary of them, and the page renders whatever the calculation stored.
- **Unknown, not zero.** If a source system can't be reached, its section is replaced by a statement that its metrics are unknown and why. Blank tiles that could be read as measured zeros are never shown. The rest of the report still renders, under a "partial report" banner.
- **The AI can fail without taking the numbers with it.** If the commentary step fails, a banner gives the reason and says the metrics are unaffected.
- **Calculate once, reuse after.** A stored report is served straight from Supabase. The workflow runs only for a new range or a forced recalculation.
- **Charts show share, not just size.** Bars are scaled to the total of their categories, so a bar's length is its share of the whole. With a single category, the value is shown without a bar.

## Limitations

- **The workflow isn't in the repo yet**, so the calculation logic can't be reviewed here. Its export is coming.
- **Configuration is hard-coded** at the top of `index.html`: the Supabase URL, the public key and the webhook URL.
- **The workflow's webhook isn't authenticated.** A token or signed request would be needed before real use.
- **The page can't see workflow errors directly.** It calls the webhook in `no-cors` mode and learns the outcome by polling. A failure shows up as a timeout after 150 seconds.
- **The data ends on 30 June 2026**, so no period can go past it.

## Run it locally

The dashboard is one static file:

```bash
python3 -m http.server 8000      # or any static server
```

Then open http://localhost:8000.

To point it at your own backend, set the three constants at the top of the script in `index.html`: `SUPABASE_URL`, `SUPABASE_ANON_KEY` and `WEBHOOK_URL`. The page expects a `report_runs` table populated by the workflow. Until the workflow export is in the repo, that table has to come from an existing deployment.

**Security:** the page uses Supabase's public anon key, which row-level security limits to reading. The workflow writes with a service-role key that never reaches the browser.
