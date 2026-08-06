---
name: pvs-line2-deploy
description: "How to reach Line 2 PVS (LINE2PVS) + its pilot deploy state — Tailscale, Administrator cred, self-contained."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-04T15:25:20.478Z
---

**Line 2 PVS deployed 2026-08-01 (monitor pilot); went FULLY LIVE 2026-08-04** — `writeProductionCount` +
`syncStockOuts` both ON, PVS is Line 2's sole DPC + balance writer (operator app on PC .126 retired, silent
since 08-01, Danial confirmed). COM map corrected to Line-1 order via Device Manager renumber + reboot (it was
a WCH card enumerating COM3/5↔4/6 opposite to Line 1, NOT bad cabling). Go-live gotcha, now fixed in code: the
startup DPC seed over-shot by ~24 boards on a MID-LOT enable (lot count still settling at the 8s seed) → the
flush now **caps each write to produced−recorded** (commit 5d39be6), which bounds any seed error to zero AND
self-heals (DPC holds until produced catches up). ⇒ For Line 5's go-live, flip the flags at a FRESH-LOT boundary
(operator=0) to avoid the artifact entirely. See [[parts-control-db-write-access]], [[pvs-line5-deploy]].

**Reaching LINE2PVS (the hard-won part):**
- Tailscale IP **100.94.102.44** (Tailscale name `line2pvs`). LAN IP 192.168.0.109 is on **Wi-Fi**, and from my laptop the LAN path to it is dead — **only Tailscale reaches it** (I reach Line 1 `.141` and Parts `.134` on the LAN fine, but not Line 2). Wire it before go-live (runbook Wi-Fi/DB-stall gate; it tested 0% loss today so not yet blocking).
- Credential: `~/line2.cred` = **`line2pvs\Administrator`** (I set + enabled the built-in Administrator; its `user` auto-login account has a blank/unknown password that Windows blocks for network logon, and there is NO `admin`/`linesvc` account — that's why every earlier guess failed). Local admins on the box: `Administrator`, `User`.
- Needed on the Line 2 PC first: `Enable-PSRemoting -Force` (WinRM was off) **and** `LocalAccountTokenFilterPolicy=1` (already set =1). `Save-Line2-Credential.bat` is on my Desktop; the cred MUST be created on my laptop `LOURDES-GUNADAS` (DPAPI-bound) — creating it on any other PC is useless.

**Deploy shape (differs from Line 1 on purpose):**
- **Self-contained** publish (`dotnet publish -c Release -r win-x64 --self-contained`, 109 MB, 421 files) copied via SMB `\\100.94.102.44\C$\PvsLineApp` — so **no .NET runtime install** was needed (Line 2 had none). Line 1 stays framework-dependent. Hot-DLL redeploys (Pvs.Core/Data/LineApp.dll) still work on both.
- `line.config.json`: `lineId 2`, `LINE2PVS`, COM map **same as Line 1** (M1 COM5, M2 COM4, M3 COM6, M4 COM3; COM1 unused — Danial confirmed), `central.server` = **192.168.0.134,1433** (LAN, faster than Tailscale here), `syncStockOuts`/`writeProductionCount` **false**. Password empty in config; `db-password.txt` (4-char pvs_ro secret) copied from Line 1 — the app reads that sidecar when config password is blank.
- Task `PvsLineApp`: `C:\PvsLineApp\Pvs.LineApp.exe --urls "http://0.0.0.0:5199"`, SYSTEM, at boot, restart 3×.

**Open items:** M4 program reads `L309MBu - B SIDE`, not a clean `L309` — verify PVS maps it to the `L309` ProductBOM before trusting a feeder-scan check on Line 2 (see [[pvs-model-naming]]). Line 2 runs L309/L311. Full go-live (DB writes) needs the operator-app + DPC handover like Line 1 ([[parts-control-db-write-access]], [[pvs-shift-and-daily-report]]). See [[schedulle-project]].
