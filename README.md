# PMO-interface
PMO Central Repository — Prototype
A working, single-file prototype of a 4-interface project portfolio system for a Government PMO. No build step, no dependencies, no server required.

Stack
Vanilla HTML + CSS + JavaScript in one file (pmo-central-repository.html)
localStorage for persistence (data survives page reloads; "Reset demo data" restores the seed)
No frameworks, no bundler — chosen so the prototype opens anywhere and can be iterated in seconds
How to run
Option A — double-click (easiest): Open pmo-central-repository.html in any modern browser (Chrome, Edge, Firefox, Safari).

Option B — local server (recommended for realistic origin behavior):


cd <folder containing the file>
python3 -m http.server 8000
Then visit http://localhost:8000/pmo-central-repository.html

Either way you get the start screen with the four interfaces. Data is stored in the browser (localStorage key pmo-prototype-data-v3), so edits persist across reloads until you click Reset demo data in the sidebar.

5-minute demo script
Governor / Executive — open it. You see only PMO-curated visuals and sector cards. Click a sector → visual drill-down by project (progress bars only — no raw tables, no finances).
Return to Interfaces (top-right), then Department → pick e.g. IT.
New Project: enter PRJ-020 + identification + first monthly status. Try typing a number into a text field or text into a number field — the review step blocks it (date/number/string type validation).
Update Existing Project: click PRJ-006, submit a monthly update; note the "Update #5" tag — it appends, it never overwrites.
Add a Budget Change Request on PRJ-006 (e.g. 3,550,000 → 3,700,000). Check the confirmed budget display: unchanged. A pending request was created.
Return to Interfaces → Financial Authority — the pending request is there. Approve → the confirmed budget is overwritten to the new value (visible back in the department interface and in Project Master).
Return to Interfaces → PMO — explore the sidebar: Overview, Sectors & Departments, Project Master (click a row for full history/risks/stakeholders), Financial/Schedule Updates, Risks & Pending (add one), Lessons Learned (try AI-style search: "vendor"), and the Visual Builder — add a visual, or flip a visual's audience to "Governor / Top Management" and watch it appear (or disappear) in the Governor interface.
Notifications: every interface has an Alerts bell in the top bar, filtered to that role only:

Governor sees new-project alerts only (no operational noise).
PMO sees every department submission (new projects, monthly updates, budget requests).
Financial Authority sees budget-request alerts.
A department only sees alerts for risks logged against its own projects (owning sector + internal stakeholders), plus approval/rejection of its budget requests. Click an alert to mark it read; "Mark all read" clears the list.
Excel export: in PMO → Schedule Updates, filter by project / sector / month, by several statuses at once (click the chips), and switch between Latest update (one row per project — pair with a month for "latest update in this month") and All updates (every monthly row), then click Download Excel. The workbook has two sheets: Projects – Latest (one row per project: full identification + the latest monthly update, with the last-updated date as the final column and an update-status flag) and Monthly Detail (the filtered monthly rows). Real .xlsx via the SheetJS CDN; falls back to .csv if offline. One shared dataset feeds every interface, so a new project shows up in all views at once.

Monthly update discipline: the prototype's "current month" is set by the CURRENT_MONTH constant (August 2026). Every open project (Active/Pending) is expected to submit an update every month; projects whose latest update is older than the current month are flagged Update due in the department home, the PMO overview, and the Project Master, and the owning department(s) automatically receive a notification to update. Closed projects (Completed/Terminated) are exempt. If a department does not update, the status is kept as-is — the flag simply persists as a visible problem. Closed projects appear in the monthly update table only through their closure row (the month they finished) — their earlier rows are dropped, so they are never part of the ongoing update flow.

Data model (9 collections)
Collection	Purpose	Written by
projects	Master record, 1 row per project ID; identification locked after creation	Department (new project), Financial Authority (confirmed budget only)
monthly	Append-only monthly status rows, numbered 1, 2, 3…	Department
budgetRequests	Proposed budget changes; only approval overwrites total_cost	Department → Financial Authority
lessons	Closure archive (Completed / Terminated projects)	Department on close
risks	Risks / pending items	PMO (optional)
visuals	PMO-curated visuals + audience tag	PMO
internal / external / team	Stakeholder junction tables	PMO
notifications	Per-role alert queue (PMO→owning dept, dept→PMO, dept→Finance, new project→Governor, Finance→dept)	all flows
Status model: a project's status is one of Active / Pending / Completed / Terminated. Delays are not a status — they are captured as risk entries (type Pending). When a department submits a monthly update with status Pending, a "pending on" reason is required, and it is automatically logged in the risks table. delay_months and revised_end remain factual metrics on the monthly row.

What to extend first (in priority order)
Real backend + auth — the single biggest gap. Replace the localStorage read/write (two functions: loadData / saveData) with a REST API (FastAPI or Express) over SQLite/Postgres. The 9 collections map 1:1 to tables, and the role picker becomes real login (Governor, Department, Financial Authority, PMO) so role separation is enforced server-side, not just in the UI.
Real AI lessons search — scoreLesson() is a keyword simulator. Swap it for an embeddings API (or a small local model) to get true semantic search over lessons + terminationReason.
Charting library for visuals — replace the CSS bar/KPI cards with Chart.js or ECharts (or embed a Power BI report per visual).
Excel/CSV import-export — departments realistically want to bulk-load monthly updates; add an export of the master + schedule updates for PMO reporting.
Notifications — email/IM alert to Financial Authority when a budget request is created, and to the department when it is approved/rejected.
Deployment — Dockerfile + a managed host so stakeholders can click a link instead of opening a file.
File layout

pmo-central-repository.html   the entire app (UI + logic + seed data)
README.md                     this file
