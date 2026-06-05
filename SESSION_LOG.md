# Session Log — Commissions Tracker POC

Status: Active

## Current state (as of 2026-06-04)

POC is complete, live on GitHub Pages at https://nabbity.github.io/comms-tracker/ and being shared with the developer for production build. Three tabs: This Month (with clickable person filter cards and month dropdown), All Projects, and Pending (with one-click FM and 2-pager approval buttons). Window logic corrected to 18th→17th to avoid double-counting. Architecture decided: approvals live in this tool, not the sales tracker. Payment data will be pulled from the sales tracker in production.

## Next up

- Await developer's production build (instructions PDF + screenshots sent 4 Jun 2026)
- Review developer's build against POC — validate trigger date logic is ported correctly
- Confirm with dev: payment data syncs from sales tracker; FM/2-pager approvals live in this tool
- Remove Edit buttons from production build (POC scaffolding only, not a real feature)

---

## Session entries

### 2026-06-04 — POC built, iterated, and handed to developer

Diagnosed the Excel logic bug: old formula used raw payment date as the window filter, so projects with late 2-pager approvals were never counted in any window. Fixed with trigger date = MAX(payment, FM date, 2-pager date), applied consistently to both DP and balance commission.

Built `index.html` — single-file web app with three tabs (This Month, All Projects, Pending). Seeded with 31 projects from the real sales tracker (May–Jun 2026 era).

Iterations made during testing session:
- Added clickable salesperson cards to filter the This Month table
- Added one-click Approve FM / Approve 2-pager buttons to the Pending tab
- Fixed window dates from 17th→17th (double-counts the boundary day) to 18th→17th
- Replaced date pickers with a month dropdown (12 months back, 3 ahead)
- Updated Pending tab description to "Not all criteria met" (more general)
- Deployed to GitHub Pages: https://nabbity.github.io/comms-tracker/

Architecture decision: FM and 2-pager approvals live in this tool. Payment data pulled read-only from the sales tracker in production. Instructions PDF + screenshots sent to developer on 4 Jun 2026 for production build.
