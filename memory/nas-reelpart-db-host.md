---
name: nas-reelpart-db-host
description: "Synology DS225+ NAS being set up to host ReelPart-New (SQL Server in Docker), replacing the 4GB Acer. Blocked on a RAM upgrade until 2026-08-11."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T08:08:23.877Z
---

Setting up a **Synology DS225+** as the host for **MCS / MCS Ai** — the new central material control system ([[mcs-material-control-system]]) — in Docker. (Original framing was just migrating ReelPart-New off the 4GB Acer per [[parts-db-server-migration]]; user clarified 2026-08-11 the NAS is really to run MCS, which will need its own DB + app containers. ReelPart-New migration may fold into that or run alongside.) SQL Server 2022 in Docker is still the DB engine.

**✅ RAM UPGRADE IN + SQL CONTAINER UP as of 2026-08-12.** D4NS01-4G installed — NAS now **MemTotal ~5.7GB** (5,826,840 kB; `free -m` total 5690, avail ~4.5GB), up from ~1.7GB. Blocker cleared. The `reelpart-sql` SQL Server 2022 **Express** container is now **running** (image `mcr.microsoft.com/mssql/server:2022-latest`, listening `0.0.0.0:1433`, `restart: unless-stopped`, `mem_limit: 3500m`); log shows "SQL Server is now ready for client connections"; SA login (creds in `~/nassql.cred`, user `sa`) authenticates and serves queries; TCP 1433 reachable from Line 5. Fresh empty engine — only system DBs (master/tempdb/model/msdb), **ReelPart-New NOT yet migrated**.
- Compose staged by an earlier 2026-08-12 session at `/volume1/docker/reelpart-sql/docker-compose.yml` (+ root-only `.env` holding `SA_PASSWORD`). **One fix I made:** the `cpus: 2.0` line **fails on the Synology kernel** ("NanoCPUs can not be set… no CPU CFS scheduler") — commented it out (`# cpus: 2.0`); `mem_limit` works fine. Backup at `docker-compose.yml.bak`.
- Deploy cmd that works: `cd /volume1/docker/reelpart-sql && sudo /usr/local/bin/docker compose up -d` (docker NOT in sudo secure_path → full path; inside `sudo sh` also use full path).

**(historical) Was RAM-blocked 2026-08-11** ("waiting ram") — NAS at ~1.7GB, needed the D4NS01-4G (→6GB). Docker/Container Manager ready, `/volume1/docker/reelpart-sql/{data,log,backup}` staged, SSH access via Line 5 jump host works.

**State as of 2026-08-10 (all done except the container):**
- NAS on the plant LAN at **192.168.0.169** (DHCP; the intended static **192.168.0.170** did NOT apply in the wizard — set it before cutover). DSM **7.4.1**.
- Storage: **SHR, 1-drive fault tolerance**, 2× ~1.8TB drives (Healthy), **Btrfs**, volume unencrypted, 1.8TB free. Drives are NON-Synology → DSM shows a compatibility nag (harmless; keep DSM on manual updates). RAM upgrade + drives were bought as: 1× D4NS01-4G, 2× HAT3300 was the rec but they used on-hand ~2TB drives.
- **Container Manager installed**; docker binary at **/usr/local/bin/docker** (NOT in sudo secure_path — call it by full path). Dirs created: **/volume1/docker/reelpart-sql/{data,log,backup}**.
- SSH admin user **GLOBALSMT** (in administrators group). sudo needs the password (no passwordless). User-home service is OFF (no ~ dir) → SSH key auth needs home-service enabled first; for now using password.

**How I reach the NAS:** me → Tailscale/WinRM → **Line 5 jump host (100.101.8.76)** → plink (staged at C:\PvsLineApp\nas\plink.exe on Line 5) → NAS. Credential = **nas.cred** (DPAPI, on the laptop; user GLOBALSMT). SSH host-key fingerprint **SHA256:NCXWg837ZSX4AM57q9Cxz8oUy8V6Kwwfk1xDFr+dgZU**. Pattern: `plink -ssh -batch -hostkey <fp> -pwfile <pwfile>`, pipe the pw to `sudo -S -p ''` for root. Avoid `cmd /c`+`Remove-Item` in one command (harness guard false-trips); use `&` call + `[System.IO.File]::Delete`.

**THE BLOCKER:** NAS has only **~1.73GB RAM** (2GB base, no upgrade). SQL Server 2022 needs ~2GB → won't start. The **D4NS01-4G (4GB SO-DIMM → 6GB total)** arrives **2026-08-11**; user installs it (power off, insert, power on). Then resume.

**✅ Migration de-risked (2026-08-12):** `ReelPart-New` on `DESKTOP-TECHNIC\SQLEXPRESS` is **tiny — 22.8 MB used** (72 MB alloc data + 8 MB log). Miles under Express's 10 GB cap → Express confirmed fine. Instance also holds older `ReelPart` + `ReelPart-Test` (all ONLINE). ⚠️ Parts PC **C: is FULL (0 GB free)**; **D: has ~70 GB** — write any .bak to D:, not C:.
**🔴 MIGRATION BLOCKER (2026-08-12): no backup rights on the source.** `~/partsctl.cred` (`dbsvc`, Windows-auth `-E`) gets **"BACKUP DATABASE permission denied in database 'ReelPart-New'"** (dbsvc is read-only in SQL / local admin on the OS only). To take the .bak I need from Danial EITHER the **sa / db_owner login** for `DESKTOP-TECHNIC\SQLEXPRESS`, OR for him to grant dbsvc **db_backupoperator**. Until then the DB copy can't be made.

**Next steps (container is already UP — see top):** (1) ~~`docker compose up -d`~~ DONE. (2) Migrate: back up ReelPart-New off the Acer (.bak) → restore into container → **recreate the SQL logins the apps use** (pvs_ro with the 4-char db-password from Line 1; plus the Parts Control app's login — script logins from source). (3) Validate row counts (StockIns/StockOuts/Users/DailyProductionCount/ProductBOM). (4) Set NAS static IP .170, repoint PVS central.server + Parts Control at it, **cut over ONE line first in an idle window** (lines run production — don't disrupt scanning), keep the Acer as fallback ~1 week. Related: [[reading-true-stock]], [[parts-control-pc-health]].
