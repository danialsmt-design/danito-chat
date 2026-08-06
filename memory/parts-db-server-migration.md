---
name: parts-db-server-migration
description: Planned move of the ReelPart-New DB off the shop-floor desktop onto a proper server; DB improvements queued for then.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-07-27T05:15:48.356Z
---

As of 2026-07-25, Danial is **getting a dedicated server ready** for the parts-control database. The ReelPart-New DB currently lives on SQL Express on the shop-floor desktop PC `DESKTOP-TECHNIC` (single point of failure; 10 GB cap; no SQL Agent). **Hold all DB-improvement work until the server is ready** — do the hardening as part of the migration/cutover, not bolted onto the current box.

Improvements queued for the migration (my advice, agreed to defer):
1. **Automated backups** — highest value; native SQL Agent jobs on the new server + off-box copy.
2. **Audit columns** (`ModifiedAt`/`ModifiedBy`) on edited tables — today's L261 confusion had no way to see what/when changed.
3. **Unique constraint** on `ProductBOM (ProductID, Machine, SupplyPosition, Side)` + **fix import to replace-not-append** — stops the corruption class (L261 A-side was 3 layered imports with duplicate feeders).
4. **Real `LineId` column** instead of encoding line in the product name (`L261` vs `L261 L3`) — see [[pvs-model-naming]].
5. Longer-term: parts master + part-number validation (typo `VS1-87-012` reached the DB), split costing (`UnitPrice`/`TotalPrice`) out of the verification BOM.

Constraints to respect: the DB is **shared with the production-count app** (not ours) — keep changes additive/backward-compatible. The CSV **import tool belongs to another person** — can supply a safe replacement but can't rewrite their app without source. Schema (DDL) needs db_owner (ACER-PC) or a db_ddladmin grant to dbsvc. See [[parts-control-db-write-access]] and [[remote-access-from-home]].

**Candidate server box (found 2026-07-27, paused for later):** `CS-001` = **192.168.0.145**, a **Dell PowerEdge R420** (service tag B8N6NW1, ~2013). Real server hardware: PERC H310 RAID controller, 2 CPU sockets (1 used: Xeon E5-2403 4c/1.8GHz), **12 DIMM slots max 384GB but only 4GB installed**, one ~930GB volume (C:464/D:2/E:464). BUT it currently runs **32-bit Windows Server 2008 SP1** (EOL, unpatched) — unusable as-is: 32-bit can't run modern SQL, capped at 4GB RAM. To repurpose as the DB server it needs: fresh 64-bit OS (Server 2019/2022 or Win11 Pro) + more RAM (cheap DDR3 ECC) + RAID-1 SSDs + verify the 930GB volume isn't a single non-redundant disk.

**⚠️ It is NOT a blank spare — it runs `Fujitrax V6.01`** (Fuji SMT traceability software) licensed by a **Sentinel USB dongle**. Before wiping, preserve it: (1) physically keep the Sentinel dongle — the license is on it, no backup captures it; (2) P2V the live box with **Disk2vhd** → bootable VHD runnable later in Hyper-V/VirtualBox with USB dongle passthrough (only ~40GB used); (3) file-level copy of the Fujitrax program+data folders over SMB (I can do this — C$/E$ reachable). User's lines are Sony, so this Fuji box is likely a retired Fuji line PC, but CONFIRM Fujitrax is dead before wiping.

**Access notes for CS-001:** WinRM would NOT enable on the 2008 box (port 5985 stays closed; `winrm quickconfig`/`Enable-PSRemoting` didn't open it). **WMI/DCOM (135) and SMB (445) DO work** with `~/cs001.cred` (`CS-001\Administrator`, new password after the old one was expired). Laptop TrustedHosts widened to `192.168.0.*,100.*`. Laptop CredSSP `AllowEncryptionOracle` was set to 2 to allow RDP to the unpatched box — remind to set back to 1.
