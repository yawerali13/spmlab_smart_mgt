# Test Cases — Smart Healthcare Project Management & Monitoring System

All test cases below were executed against the actual application (not just described
on paper). "Actual Result" values were captured from a real run of the system with the
seeded case-study data, as-of date **2026-02-20** (the default), before any manual edits
were made in that session. Re-running the steps on a freshly reset dataset reproduces
the same figures, which is itself part of Test Case TC-14.

---

### Module 1 — Project Initiation & Integration Management

**TC-01: Create / view project charter**
- Steps: Open the app → Integration tab.
- Expected: Charter shows name, objectives, stakeholders, scope boundaries, approved
  budget PKR 5,000,000, deadline 16 weeks, start date 2026-01-05.
- Actual: Matches exactly as seeded. **PASS**

**TC-02: Baseline status reflects approved change requests**
- Steps: Integration tab → read "Baseline Status (computed)".
- Expected: Current Approved Budget = 5,000,000 + 800,000 (approved AI/mobile CR) =
  PKR 5,800,000. Current Deadline = start date + 16 weeks + 21 days (approved CR
  schedule impact) = 2026-05-18.
- Actual: Current Approved Budget shown as **PKR 5,800,000**; Current Deadline shown
  as **2026-05-18**. **PASS**

**TC-03: Reject invalid charter save (empty required field)**
- Steps: Clear the "Project Name" field → Save Charter.
- Expected: Browser's native "required" validation blocks submission; no DB row is
  corrupted.
- Actual: Form does not submit until Project Name is filled. **PASS**

**TC-04: Add / delete milestone**
- Steps: Add milestone "UAT Sign-off", due 2026-05-10 → verify row appears → delete it.
- Expected: Row appears immediately, then disappears after delete (confirmation
  prompt shown first).
- Actual: Row appeared and was removed after confirming the delete prompt. **PASS**

---

### Module 2 — Scope Management & WBS

**TC-05: Unapproved requirement cannot enter the WBS**
- Steps: Scope tab → Add WBS Node → attempt to select a "Proposed" requirement
  (e.g. "AI-based preliminary diagnosis…") in the requirement dropdown.
- Expected: The requirement dropdown only lists **Approved** requirements, so a
  Proposed one cannot be selected at all; if the underlying guard is bypassed
  (e.g. by manipulating the form), `addWbs()` re-validates server-side and shows
  "Only approved requirements can be linked into the WBS."
- Actual: Dropdown contained only the 5 Approved requirements (REQ-1..REQ-5); the
  2 Proposed requirements (AI diagnosis, mobile app) were absent. **PASS**

**TC-06: Approve a requirement, then link it to a new WBS node**
- Steps: Approve "AI-based preliminary diagnosis…" (REQ-6) → Add WBS node
  "1.6 AI Diagnosis Integration" linked to REQ-6.
- Expected: REQ-6 status becomes Approved; new WBS node appears under the tree with
  "(REQ-6)" tag.
- Actual: Status badge changed to **Approved**; node "1.6 AI Diagnosis Integration
  (REQ-6)" rendered in the WBS tree. **PASS**

**TC-07: Change request approval updates cost & schedule baseline**
- Steps: Scope tab → approve CR-2 ("Add SMS appointment reminders", cost impact
  PKR 60,000, schedule impact 4 days).
- Expected: Integration tab's Current Approved Budget rises to PKR 5,860,000 and
  Current Deadline shifts 4 days later (2026-05-22); Dashboard's "Budget Utilized"
  denominator updates to match.
- Actual: Verified — approving the CR immediately recalculated both figures on the
  Integration and Dashboard tabs (computed live from `SUM(cost_impact) WHERE
  status='Approved'`, not cached). **PASS**

**TC-08: Add change request with non-numeric cost impact**
- Steps: Submit the "Submit CR" form with the cost impact field left blank.
- Expected: HTML5 `required` + `type=number` validation blocks submission.
- Actual: Form did not submit; no row inserted. **PASS**

---

### Module 3 — Schedule & Resource Management

**TC-09: Reject activity where planned end precedes planned start**
- Steps: Schedule tab → Add Activity "Bad activity", start 2026-03-01,
  end 2026-02-01, cost 1000 → submit.
- Expected: Error banner "Planned end date cannot be before start date"; no row
  inserted.
- Actual: Error banner displayed with exactly that message; activity list unchanged
  until a corrected date was supplied. **PASS** (confirmed via automated browser test)

**TC-10: Delayed-activity detection**
- Steps: Schedule tab, as-of date = 2026-02-20 (default).
- Expected: Any activity whose planned end date is before the as-of date and is not
  "Completed", or whose actual end date is after its planned end date, is flagged
  "Delayed".
- Actual: 4 activities flagged Delayed: "Design appointment booking workflow"
  (finished 2 days late), "Build booking & reschedule/cancel APIs" (planned end
  2026-02-13 has passed, still In Progress), "Build lab report upload & viewing"
  (finished 4 days late), "Build doctor duty roster management" (planned end
  2026-01-30 has passed, still In Progress). **PASS**

**TC-11: Dependency cascade impact**
- Steps: Activities "1.1.2 Build booking & reschedule/cancel APIs" (depends on
  1.1.1) and "1.1.3 Test appointment scheduling module" (depends on 1.1.2).
- Expected: Because 1.1.1 finished only 2 days late (2026-01-18 vs planned
  2026-01-16) and 1.1.2's planned start (2026-01-19) already had slack, the
  cascade should absorb the delay without pushing the final deadline.
- Actual: "✔ No cascading slip to the final deadline detected" shown, with
  Projected Deadline Slip = **0 days**. Changing 1.1.1's actual end to
  2026-01-25 (manually, for verification) correctly produced a non-zero
  "Delayed +Nd" tag that propagated to 1.1.2 and 1.1.3. **PASS**

**TC-12: Mark activity 100% complete**
- Steps: Set an In Progress activity's % Complete field to 100.
- Expected: Status auto-changes to "Completed"; actual_end is set to the current
  as-of date if not already set.
- Actual: Status dropdown updated to Completed automatically after the % field
  changed. **PASS**

**TC-13: Gantt chart renders without error for all 10 seeded activities**
- Steps: Open Schedule tab, inspect the Gantt section.
- Expected: One bar per activity, positioned by planned start/end, coloured by
  status, with a dashed red line at the as-of date; no JavaScript console errors.
- Actual: 10 bars rendered correctly (automated headless-browser run captured
  zero console/page errors across all 6 tabs). **PASS**

---

### Module 4 — Cost Management & Earned Value Analysis

**TC-14: EVM indicators match hand-calculated values (as of 2026-02-20)**
- Expected (hand-calculated from the seeded activity data using
  PV=Σ(cost_estimate × elapsed_fraction), EV=Σ(cost_estimate × %complete),
  AC=Σ(actual_cost)):
  - PV ≈ PKR 1,512,545
  - EV = PKR 1,334,500
  - AC = PKR 1,433,000
  - CPI = EV/AC ≈ 0.93
  - SPI = EV/PV ≈ 0.88
- Actual (read directly from the running application): PV = **PKR 1,512,545**,
  EV = **PKR 1,334,500**, AC = **PKR 1,433,000**, CPI = **0.93**, SPI = **0.88**.
  **PASS — exact match**

**TC-15: Automatic threshold warning**
- Steps: Cost tab, same as-of date.
- Expected: Because SPI (0.88) < 0.90, a warning message is displayed; because
  CPI (0.93) ≥ 0.90, no cost-overrun warning is shown for CPI alone.
- Actual: Displayed exactly: "⚠ Schedule Performance Index (SPI = 0.88) is below
  0.90 — the project is behind schedule relative to the plan." No CPI warning
  shown. **PASS**

**TC-16: Updating actual cost recalculates AC/CPI live**
- Steps: Increase "Build invoicing & payment tracking" actual cost from
  PKR 220,000 to PKR 400,000.
- Expected: AC increases by PKR 180,000 to PKR 1,613,000; CPI drops below 0.93;
  if it falls under 0.90 a cost-overrun warning appears.
- Actual: AC recalculated to **PKR 1,613,000**, CPI dropped to **0.83**, and the
  cost-overrun warning appeared alongside the existing SPI warning. **PASS**

---

### Module 5 — Quality Management & Risk Control

**TC-17: Critical defects visible and linked to activities**
- Steps: Quality tab → Defect Tracking.
- Expected: 4 seeded defects shown, including 2 "Critical" severity items linked
  to "Build booking & reschedule/cancel APIs" and "Build lab report upload &
  viewing", matching the case study's note that defects were found in appointment
  scheduling and patient record management.
- Actual: Both Critical defects present and correctly linked. **PASS**

**TC-18: Risk prioritization matrix placement**
- Steps: Quality tab → inspect the 5×5 matrix.
- Expected: "Integration delay between hospitals' legacy systems…" (Likelihood 4,
  Impact 5, score 20) appears in the top band (red, score ≥ 15) and ranks #1 in
  the risk table.
- Actual: Risk appeared in the red cell at (Likelihood 4, Impact 5) and was listed
  first in the sorted risk table with score **20**. **PASS**

**TC-19: PDPC view shows preventive and corrective actions**
- Steps: Quality tab → scroll to Process Decision Program Chart section.
- Expected: Each risk renders as a card with a Preventive Action box and a
  Corrective Action box.
- Actual: All 5 seeded risks rendered with both boxes populated (e.g. the data
  privacy risk shows "Enforce end-to-end encryption…" preventive and "Activate
  incident response plan…" corrective). **PASS**

**TC-20: Quality checklist records a failed inspection**
- Steps: Quality tab → Quality Checklist.
- Expected: "Lab report access restricted to assigned doctor and patient" shows
  result = Fail.
- Actual: Row shown with red "Fail" badge. **PASS**

---

### Module 6 — System Integration, Validation & Reporting

**TC-21: Dashboard aggregates all modules correctly**
- Steps: Open Dashboard tab (as-of 2026-02-20, before any manual edits).
- Expected: Overall Progress = EV/BAC = 1,334,500 / 2,320,000 ≈ 58%; Pending
  Change Requests = 1; Delayed Activities = 4; Unresolved Critical/High Defects
  = 3 (2 Critical still Open/In Progress + 1 High Open; the Low-severity
  resolved defect is excluded); CPI = 0.93; SPI = 0.88.
- Actual: Dashboard displayed **58%**, **1** pending CR, **4** delayed activities,
  **3** unresolved critical/high defects, CPI **0.93**, SPI **0.88** — all
  matching the hand-calculated expectation exactly. **PASS**

**TC-22: Change propagates across modules (integration test)**
- Steps: On Scope tab, approve the pending "SMS reminders" CR (PKR 60,000 / 4
  days) → check Integration tab and Dashboard.
- Expected: Both tabs immediately reflect the new current budget (PKR 5,860,000)
  without a page reload, proving the six modules share one underlying database
  rather than independent, disconnected screens.
- Actual: Confirmed — both tabs updated instantly after approval. **PASS**

**TC-23: Data persists across a page reload**
- Steps: Add a new requirement, reload the browser tab (F5).
- Expected: The new requirement is still present after reload (loaded from the
  SQLite binary stored in `localStorage`).
- Actual: Requirement persisted after reload. **PASS**

**TC-24: "Reset Sample Data" restores a clean, consistent dataset**
- Steps: After making several edits, click "Reset Sample Data" and confirm.
- Expected: All tables are cleared and re-seeded with the original case-study
  data; Dashboard figures return to the TC-21 baseline.
- Actual: Dashboard returned to the original baseline figures after reset
  (automated test confirmed the tab re-rendered without error). **PASS**

**TC-25: No unhandled JavaScript errors across the full application**
- Steps: Automated headless-browser run through all 6 tabs plus the interactions
  in TC-05 through TC-24.
- Expected: Zero console/page errors.
- Actual: **Zero errors recorded** across the full automated test session.
  **PASS**

---

## Summary

| # | Test Case | Result |
|---|---|---|
| TC-01 | View charter | PASS |
| TC-02 | Baseline reflects approved CR | PASS |
| TC-03 | Reject empty charter name | PASS |
| TC-04 | Add/delete milestone | PASS |
| TC-05 | Unapproved requirement excluded from WBS | PASS |
| TC-06 | Approve requirement → link to WBS | PASS |
| TC-07 | CR approval updates budget/schedule | PASS |
| TC-08 | Reject non-numeric CR cost | PASS |
| TC-09 | Reject invalid activity dates | PASS |
| TC-10 | Delayed-activity detection | PASS |
| TC-11 | Dependency cascade impact | PASS |
| TC-12 | Auto-complete at 100% | PASS |
| TC-13 | Gantt chart renders cleanly | PASS |
| TC-14 | EVM indicators match hand calculation | PASS |
| TC-15 | Automatic threshold warnings | PASS |
| TC-16 | Live AC/CPI recalculation | PASS |
| TC-17 | Critical defects tracked | PASS |
| TC-18 | Risk matrix placement | PASS |
| TC-19 | PDPC view | PASS |
| TC-20 | Failed inspection recorded | PASS |
| TC-21 | Dashboard aggregation | PASS |
| TC-22 | Cross-module integration | PASS |
| TC-23 | Persistence across reload | PASS |
| TC-24 | Reset to sample data | PASS |
| TC-25 | No unhandled errors | PASS |

**25 / 25 test cases passed.**
