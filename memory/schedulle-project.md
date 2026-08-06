---
name: schedulle-project
description: "Schedulle team check-in tracker — architecture, deployment, and key config"
metadata: 
  node_type: memory
  type: project
  originSessionId: 1273fda6-0c2d-47eb-b01f-00d58d2a986e
  modified: 2026-08-06T02:55:26.092Z
---

## VERSION NAMING (user's convention, adopted 2026-07-16)
- **v1.0** = ORIGINAL version: engineer list in `config.js` (Aiezzat/Firman/affeq),
  old `db.js` (no departments/staff/accounts), shared dashboard password, old agent
  (picks name from a short list). **THIS IS WHAT'S LIVE** on farid's PC at
  `C:\Users\farid\Desktop\schedulle-server-bundle`, with real data. v1.0 source now
  exists ONLY there — this project folder was rewritten to v2.0. Worth backing up.
- **v2.0** = COMPANY-WIDE version built here (departments/staff/accounts, logins,
  admin console, agent auto-identify by Windows login). NOT yet deployed.

⚠️ v1 and v2 are NOT wire-compatible: the v2 server has no `/api/config` (404) and
rejects old-style `{engineer: name}` check-ins/heartbeats (409). Never mix a v1
agent with a v2 server, or copy v2 files onto a v1 server — that caused
"TypeError: db.listDepartments is not a function".

Zips: `Schedulle-v2.0-Complete-Setup.zip` (for SL Lee),
`Schedulle-v2.0-server-bundle.zip`, `Schedulle-v2.0-Engineer-App.zip`,
`Schedulle-REPORT-FIX-works-on-v1-and-v2.zip` (self-contained daily-report.js +
.bat; auto-detects v1/v2; opens DB read-only).

## LIVE v1.0 server + monitoring (2026-07, farid's PC)
v1.0 runs on **desktop-pua4prr**, Tailscale IP **100.66.246.104**, port 3000,
folder `C:\Users\farid\Desktop\schedulle-server-bundle`. As of last check the PC
was online (ping ok) but **port 3000 was NOT reachable over Tailscale** — almost
certainly the firewall rule was local-network-only (Tailscale = Public profile),
possibly also not auto-starting. Tailscale SSH is OFF on that PC (can't remote-exec).
Fix built in `WATCHDOG/` (+ `Schedulle-WATCHDOG.zip`): `fix-and-autostart.bat`
(run on farid's PC — opens firewall port 3000 -Profile Any, installs the
"Schedulle Server" auto-start service, starts it) and `watchdog.sh` (runs on the
Pi via cron */5, curls the server, WhatsApps on down + on recovery, no spam).
WhatsApp send = the same whatsapp-mcp bridge, POST http://localhost:8080/api/send
{"recipient":"60122185237","message":...} — VERIFIED working end-to-end. Alert
channel confirmed (user received tests). [[gmail-whatsapp-digest]]

Schedulle (in `Downloads/schedulle`) tracks 3 engineers' 2-hourly work check-ins.
Two parts:

- **Server**: Node/Express + better-sqlite3, runs on one office PC (LAN IP
  `192.168.100.36:3000`). Serves the manager dashboard (`/dashboard.html`,
  password `manager123`) and API. DB is `checkins.db` (tables: `checkins`,
  `status`). All tunables live in `config.js` (names, CHECKIN_TIMES,
  work hours/days, IDLE_THRESHOLD_MIN=5, password).
- **Desktop agent**: `agent/checkin_agent.py` → compiled to
  `dist/SchedulleCheckin.exe` (PyInstaller, onefile/noconsole). Windows tray app,
  auto-starts on login via `dist/install.bat`. Tracks mouse/keyboard idle via
  Win32 `GetLastInputInfo`, pops a check-in form each slot, POSTs check-ins +
  `/api/heartbeat`. Agents pull schedule/idle-threshold from `/api/config` at
  startup, so config.js changes don't need an exe rebuild (only server restart +
  engineer re-login).

Build venv is `agentbuild/` (Python 3.14).
User (manager) is non-technical — prefers concrete URLs, double-click .bat flows.
Idle/activity monitoring was requested; team-transparency was flagged as good
practice. [[user-danial-schedulle]]

## Scaling to company-wide (~160 staff, 27 depts) — Phase 1 DONE
Decisions: stays on an office PC; individual manager/admin logins; manual per-PC
install; idle tracking kept (user says approved). Staff identity = self-register
on first run (roster has no Windows-login column). Kept all 27 raw dept labels
from the spreadsheet as-is (user chose not to consolidate).

Roster source: `C:\Users\...\Downloads\DAILY ATTN 2025.xlsx`, sheet "SEP 25",
cols ID NO./NAME/DEPT → 154 rows, deduped to 150 staff. Regenerate CSV with the
openpyxl snippet, then `node import-staff.js staff.csv`.

Phase 1 built + tested: new schema (departments, staff w/ employee_id + nullable
windows_login, accounts, account_departments, sessions; checkins/status keyed by
staff_id). `auth.js` = scrypt hashing + cookie sessions (schedulle_session, 12h).
CLIs: `import-staff.js`, `create-account.js`. Agent rewritten to identify by
Windows login (getpass.getuser) via /api/agent/identify → self_register dialog →
/api/agent/register binds login→staff. Dashboard is now login-gated
(login.html), department-grouped, scoped by role (admin=all, manager=own depts).
Test accounts exist with weak "ChangeMe-*" passwords — replace before real use.
exe rebuilt.

## Phase 2 DONE — admin console + roll-up dashboard
`admin.html`/`admin.js` (admin-only, role-gated): tabs for Staff (search/filter/
add/edit/unbind login), Departments (add/rename/edit per-dept schedule), Accounts
(create/edit/reset-pw/delete, dept scoping). Backed by `/api/admin/*` routes +
requireAdmin. Dashboard now uses collapsible <details> per department (expanded
only when <=3 depts) with summary header (people/online/%done/blocked) + global
overview tiles; admin sees an "Admin" link.

Data-quality artifacts in the roster the admin should clean via UI: a junk row
named "NAME" in dept "DEPT" (spreadsheet header imported as a person), and
"NO.1" looks like a placeholder dept (22 people). Server bundle zip is now STALE
(predates org-mode) — refresh before deploying.

## Phase 3 DONE — reports & email
`report.js` (org-aware text builder), `daily-report.js` (exports generateFiles();
writes reports/<date>/company.txt + one per dept), `mailer.js` (nodemailer,
OFF unless config.SMTP.host set), `send-reports.js` (saves files + emails each
account its scoped report: admin=company, manager=own depts), `test-email.js`.
Accounts gained an `email` column (admin UI + endpoints). SMTP block added to
config.js (blank = off, graceful). Scheduled task (install-report-schedule.bat)
now runs send-reports.js at 18:00. Targeting verified via jsonTransport.

## Phase 4 DONE — packaging & rollout
All 3 bundles REFRESHED with org-mode files (schedulle-server-bundle.zip,
Schedulle-Engineer-Setup.zip, Schedulle-Complete-Setup.zip). setup-server.bat
now installs deps only + prints next steps (import/create-account/service).
SERVER-SETUP.txt, engineer READ ME, START HERE all rewritten for org flow
(import staff.csv, create-account, self-register agent, login dashboard).
install.bat gained /silent mode for GPO/Intune push. sign.bat + SIGNING.txt =
code-signing tooling (user must buy a cert; can't sign without it). Report
schedule now runs send-reports.js. staff.csv (real 150-person roster) shipped in
bundles. CLEAN-ROOM TESTED: unzip → npm install → import → create-account →
start → login/dashboard/agent identify all pass.

ALL 4 PHASES COMPLETE. Remaining/optional: buy code-signing cert; consider HTTPS
(currently plain HTTP on LAN); clean roster artifacts ("NAME"/"DEPT" junk row,
"NO.1" placeholder dept) via Admin UI. Deployable bundles are current.
