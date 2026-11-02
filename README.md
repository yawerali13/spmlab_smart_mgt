# Smart Healthcare Project Management & Monitoring System

A complete, working Software Project Management System built for the Lab MidTerm
case study: a 5-hospital network development project (appointments, lab reports,
doctor schedules, online consultations, billing), including the mid-project
scope change (AI diagnosis + mobile app, +PKR 800,000, +3 weeks) and the
appointment-scheduling / patient-record defects described in the brief.

## How to run it

**Option A — single file (simplest):** open `healthcare_pms.html` directly in any
modern browser (Chrome, Edge, Firefox). That's it — no install, no server. An
internet connection is required the first time so the browser can download the
SQLite-in-WebAssembly engine (`sql.js`) from a CDN; after that the app and its
data work entirely locally.

**Option B — source folder:** open `index.html` (same folder as `app.js` and
`styles.css`) the same way. This is the same application split into its
source files for readability/grading.

No build step, server, or database installation is required — the application
runs entirely client-side and uses a **real SQLite database** (via `sql.js`,
SQLite compiled to WebAssembly) persisted in the browser's local storage.

## What's inside

| File | Purpose |
|---|---|
| `healthcare_pms.html` | Self-contained single-file build (HTML+CSS+JS inlined) — the "executable application" |
| `index.html`, `app.js`, `styles.css` | Same application as separate source files |
| `schema.sql` | Standalone copy of the database schema (10 tables) |
| `test_cases.md` | 25 test cases with expected and actual results, executed against a real running instance |
| `README.md` | This file |

## Sample / seed data

On first run the app automatically seeds the database with realistic data
matching the case study:
- The project charter (PKR 5,000,000 budget, 16-week deadline, 5 hospitals)
- 7 team members (PM, 2 backend, 2 frontend, 1 DB developer, 1 QA — exactly the
  team composition given in the brief)
- 7 requirements (5 Approved: appointments, lab reports, doctor schedules,
  consultations, billing; 2 still Proposed: AI diagnosis, mobile app — to
  demonstrate the scope-approval gate)
- A 3-level WBS (Project → 5 Deliverables → 11 Work Packages)
- 2 change requests: the case study's AI-diagnosis + mobile-app request
  (PKR 800,000 / +21 days, **Approved**) and an SMS-reminder request (Pending)
- 10 activities with realistic planned vs. actual dates/costs/% complete,
  including finish-to-start dependencies
- 4 defects, including the 2 **Critical** ones in appointment scheduling and
  patient record management called out in the case study
- 4 quality-checklist inspection results
- 5 risks with likelihood/impact scores and PDPC preventive/corrective actions
- 5 milestones

Use the **"Reset Sample Data"** button at any time to wipe all edits and
restore this exact dataset — useful for re-testing or re-grading.

## How each module maps to the rubric

1. **Project Initiation & Integration Management** — "1. Initiation &
   Integration" tab: charter fields, the 4-part management plan (scope/
   schedule/cost/quality), milestones, status, and a closure section. The
   "Baseline Status" panel is computed live (approved budget + sum of
   approved CR cost impacts; deadline + sum of approved CR schedule impacts).
2. **Scope Management & WBS** — "2. Scope & WBS" tab: requirements with
   Proposed/Approved/Rejected workflow, a hierarchical WBS tree, and change
   requests with cost/schedule impact and an approval workflow. Unapproved
   requirements are structurally excluded from the WBS "link to requirement"
   dropdown and re-validated server-side in `addWbs()`.
3. **Schedule & Resource Management** — "3. Schedule & Resources" tab:
   activities with assignee, dates, dependencies, a delayed-activity detector,
   a dependency cascade calculator (`computeScheduleAnalysis()`), and an
   HTML/CSS Gantt-style timeline with an as-of-date marker.
4. **Cost Management & EVM** — "4. Cost & EVM" tab: PV/EV/AC/CPI/SPI/CV/SV/EAC
   computed programmatically per activity and aggregated
   (`computeEVM()`), with automatic warning banners when CPI or SPI drops
   below 0.90 or AC/EAC exceeds the current approved budget.
5. **Quality Management & Risk Control** — "5. Quality & Risk" tab: defect
   tracker, deliverable inspection checklist, a 5×5 likelihood×impact risk
   prioritization matrix (color-coded), and a PDPC-style card view showing
   each risk's preventive and corrective actions.
6. **System Integration, Validation & Reporting** — the Dashboard tab
   aggregates live data from every module above (progress, pending CRs,
   delayed activities, budget utilisation, unresolved critical defects, CPI/
   SPI, projected deadline slip) into one project status report, all read
   from the single shared SQLite database.

## Validation & exception handling

- Required-field and numeric/date validation on every form (HTML5 plus
  JavaScript checks, e.g. "planned end before planned start" is rejected).
  Errors surface as a dismissible banner rather than crashing the page.
  A **governing "database for anyone" scope-control rule** is enforced in code:
  `addWbs()` refuses to link a WBS node to a requirement that is not Approved.
- All `db.run` / `db.exec` calls are wrapped in `try/catch`; failures show a
  banner instead of leaving the UI in an inconsistent state.
- Deleting a record always asks for confirmation first.

## Demonstrating / screenshotting the system

Because this is a browser app, screenshots for the submission can be taken by
simply opening `healthcare_pms.html`, clicking through the six tabs, and using
your OS's screenshot tool — no special capture setup is needed. The
"as-of date" control at the top lets you move the simulated "today" forward or
backward to demonstrate how delayed-activity detection and EVM indicators
change over time.

## Technical notes

- Database: SQLite via `sql.js` (WebAssembly build of SQLite), loaded from a
  CDN at startup. All records (project, requirements, WBS, change requests,
  activities, dependencies, defects, checklist items, risks, milestones, team
  members) live in 10 real SQL tables — see `schema.sql`.
- Persistence: after every write, the database is serialized
  (`db.export()`) and stored as base64 in `localStorage`, then reloaded on
  the next visit — so data survives reloads and browser restarts.
- No external backend/server is required; everything runs client-side.
- Tested with an automated headless-browser pass through all 6 modules and
  25 scenarios (see `test_cases.md`) with zero unhandled JavaScript errors.
