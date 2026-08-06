---
name: pvs-line5-deploy
description: Reaching Line 5 PVS (LINE5PVS) + its monitor-only pilot deploy; COM swap to Line-1 order is pending.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-03T05:58:54.061Z
---

**Line 5 PVS deployed 2026-08-03 as a monitor-only pilot** (app + DB OK; NO writes). Same recipe as [[pvs-line2-deploy]].

**Reaching LINE5PVS:**
- Tailscale **100.101.8.76** (name `line5pvs`). LAN IP **192.168.0.109**, and its LAN path to the Parts PC works
  (LAN_DB ok), so config uses `central.server = 192.168.0.134,1433` (LAN, faster than Tailscale). NOTE Line 5's
  .109 is the same LAN IP Danial first gave for Line 2 — Line 2 was actually reached via Tailscale only, so watch
  for a possible .109 address clash between those two PCs.
- Credential `~/line5.cred` = **`line5pvs\Administrator`** (the `Prepare-Line5-PC.bat` desktop helper enabled the
  Administrator + set its password; WinRM + `LocalAccountTokenFilterPolicy=1` done by the same bat). The bare
  `admin`/`user` accounts don't work remotely — Administrator is the one.

**Deploy:** self-contained build (`publish/line2`, reused — line-agnostic) copied via SMB `\\100.101.8.76\C$\PvsLineApp`;
`line.config.json` lineId 5 / LINE5PVS / pilot flags off / db-password.txt copied from Line 1; task `PvsLineApp`
= `Pvs.LineApp.exe --urls http://0.0.0.0:5199`, SYSTEM, at boot. Desktop shortcut `C:\Users\Public\Desktop\PVS.url`
→ http://localhost:5199/. Line 5 runs **A-side** (L307/L313; L313 has no Line-5 ProductBOM — gap).

**⚠ OPEN — COM mapping:** at deploy only **COM3 answered, and it is `Cell1`** (Line 1's COM3 = Cell4); COM4/5/6 were
silent. Config keeps the Line-1 order (M1 COM5, M2 COM4, M3 COM6, M4 COM3). **Danial will physically SWAP the Line 5
serial cables to match Line 1 later** — after that all four come online and M4 = last cell with no config change.
Until swapped, machines won't stream (fine in pilot). Full go-live (DB writes) needs the operator-app/DPC handover
like Line 1. See [[schedulle-project]], [[parts-control-db-write-access]].
