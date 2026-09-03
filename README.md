# Therese — Week 2 Ops Reporting Dashboard

Operations reporting dashboard for Koya Talent, built for Week 2 of the AI Automation Academy.

**Live:** https://therese-week2-ops-reporting.vercel.app

## What this is

A single static page that reads from Supabase and displays operating metrics for a
selected reporting period across Sales, Project Delivery and People Ops, together with
AI-generated insights and data quality warnings.

The reports themselves are produced by an n8n workflow (`Therese — Week 2 Ops Reporting`)
which pulls from Google Sheets, Airtable and an internal People Ops API, calculates the
metrics deterministically, sends a summary to Claude for interpretation, and stores the
result in Supabase. This page only reads and displays; it never calculates.

All periods are anchored to a reference date of **30 June 2026**, because the source
data ends there.

## Security

The page uses Supabase's public anon key, restricted to read-only by row-level security.
Writes are performed by the n8n workflow using a service-role key that is never exposed
to the browser.
