---
name: mcs-cutover-readiness
description: State of the ReelPart-New → NAS cutover — what's ready, the app inventory that must repoint, and the open items. Paused 2026-08-12 eve.
metadata:
  node_type: memory
  type: project
---

Cutover of **ReelPart-New** from the Parts PC (`DESKTOP-TECHNIC` / `192.168.0.134` / SQLEXPRESS) to the **NAS** container (`192.168.0.169,1433`, also Tailscale `100.125.22.119`). See [[nas-reelpart-db-host]] (NAS/DB ready, nightly backups, static IP, bond, Tailscale) and [[mcs-material-control-system]]. **Paused 2026-08-12 eve — "continue later."**

**✅ DECISION 2026-08-12: DON'T cut over the lines yet.** Leave line PCs pointing at `.134`. Reason below (split-brain) — the PC3 writers must be ready to move at the same time. Working on PC3 next.

**⚠️ SPLIT-BRAIN RULE:** ReelPart-New has multiple writers; ALL must point at ONE DB or the copies diverge. Writers:
- **Line PCs (PVS)** — connect as `pvs_ro` (SQL auth). `pvs_ro` = db_datareader + explicit `GRANT INSERT ON DailyProductionCount`, `GRANT UPDATE ON StockOuts`. Config: `C:\PvsLineApp\line.config.json` → `central.server="192.168.0.134,1433"` (change to `192.168.0.169,1433`), then restart scheduled task **`PvsLineApp`** (`C:\PvsLineApp\Pvs.LineApp.exe`). All 5 lines identical. Lines write DailyProductionCount + StockOuts.
- **Parts Control desktop app (goods-in / StockIns)** — runs on **PC3 (.134)** only (desktop shortcut `PartsControlSystem_App- 7-30-26 - Shortcut` → **`D:\Release Parts Control-GMS\PartsControlSystem_App- 7-30-26.exe`**, a WinForms app; folder has many dated hand-built exes). **✅ Deployed exe AND current source use SQL AUTH** (NOT Windows auth — the `Integrated Security=True` seen in the older "Sony F130 parts control system" session snippet + `SQL_Handler - 22012026.cs` was a STALE version). Exact conn (verified in the running exe's UTF-16 strings + source):
  `Data Source=192.168.0.134\SQLEXPRESS; Initial Catalog=ReelPart-New; User Id=partcontrol_user; Password=GMS2026;`
  Source: **`D:\WindowsForms_CustomTab\SQL_Handler.cs`** `getConnection()` (last mod 8/6/2026). VS2015 is installed on PC3 (it's the dev box). **partcontrol_user/GMS2026 already exists on the NAS** and **PC3→NAS with those creds is verified** (StockIns=10002). → **Repoint = change Data Source `192.168.0.134\SQLEXPRESS` → `192.168.0.169,1433` only.** Options: (a) **SQL client alias** on PC3 (`192.168.0.134\SQLEXPRESS`→`192.168.0.169,1433`, no rebuild, reversible — works now BECAUSE it's SQL auth), (b) edit `SQL_Handler.cs` + rebuild in VS2015, (c) externalize conn to `app.config` + rebuild (future-proof). No Windows-auth blocker.

**🔎 STILL OPEN (next session):**
- **ProductionAPI** (IIS on PC3, app pool `ProductionAPIPool`, path `D:\PRODUCTION COUNT\ProductionAPI\ProductionAPI\bin`) — the web app **PC1 (.112) / PC2 (.121) browse to** (production count). Its conn string is **NOT in Web.config / ProductionAPI.dll.config** (only VS template placeholder `ReleaseSQLServer`/`MyReleaseDB`) → likely hard-coded in `ProductionAPI.dll` or built in code. **Determine what it connects with and whether it WRITES** (if it writes, it's a 3rd writer that must repoint too). Nothing was connected to SQL locally at check time.
- Decide repoint method for the Parts Control app (alias vs rebuild vs config-externalize).
- Then a coordinated full cutover in an idle window: fresh backup → restore over NAS staging copy → flip lines + PC3 app (+ ProductionAPI) together → validate → keep `.134` as fallback ~1 week.

**Client/endpoint inventory (Danial 2026-08-12):** 5 line PCs (.105/.147/.126/.144/.157) + PC1 .112 + PC2 .121 + PC3 .134 (DB host) + Danial's PC. PC1/PC2 = browser clients of the web app (no direct repoint if only the web app's conn moves). All lines reach the NAS on both LAN (.169:1433) and Tailscale — verified; line2/line3 route over USB WiFi dongles (Danial: leave as-is).
