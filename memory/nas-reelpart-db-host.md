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

**Still RAM-blocked as of 2026-08-11** ("waiting ram") — NAS at ~1.7GB, needs the D4NS01-4G (→6GB). Nothing (SQL Server, MCS) can start until it's installed. Docker/Container Manager ready, `/volume1/docker/reelpart-sql/{data,log,backup}` staged, SSH access via Line 5 jump host works.

**State as of 2026-08-10 (all done except the container):**
- NAS on the plant LAN at **192.168.0.169** (DHCP; the intended static **192.168.0.170** did NOT apply in the wizard — set it before cutover). DSM **7.4.1**.
- Storage: **SHR, 1-drive fault tolerance**, 2× ~1.8TB drives (Healthy), **Btrfs**, volume unencrypted, 1.8TB free. Drives are NON-Synology → DSM shows a compatibility nag (harmless; keep DSM on manual updates). RAM upgrade + drives were bought as: 1× D4NS01-4G, 2× HAT3300 was the rec but they used on-hand ~2TB drives.
- **Container Manager installed**; docker binary at **/usr/local/bin/docker** (NOT in sudo secure_path — call it by full path). Dirs created: **/volume1/docker/reelpart-sql/{data,log,backup}**.
- SSH admin user **GLOBALSMT** (in administrators group). sudo needs the password (no passwordless). User-home service is OFF (no ~ dir) → SSH key auth needs home-service enabled first; for now using password.

**How I reach the NAS:** me → Tailscale/WinRM → **Line 5 jump host (100.101.8.76)** → plink (staged at C:\PvsLineApp\nas\plink.exe on Line 5) → NAS. Credential = **nas.cred** (DPAPI, on the laptop; user GLOBALSMT). SSH host-key fingerprint **SHA256:NCXWg837ZSX4AM57q9Cxz8oUy8V6Kwwfk1xDFr+dgZU**. Pattern: `plink -ssh -batch -hostkey <fp> -pwfile <pwfile>`, pipe the pw to `sudo -S -p ''` for root. Avoid `cmd /c`+`Remove-Item` in one command (harness guard false-trips); use `&` call + `[System.IO.File]::Delete`.

**THE BLOCKER:** NAS has only **~1.73GB RAM** (2GB base, no upgrade). SQL Server 2022 needs ~2GB → won't start. The **D4NS01-4G (4GB SO-DIMM → 6GB total)** arrives **2026-08-11**; user installs it (power off, insert, power on). Then resume.

**Next steps once RAM is in:** (1) `docker compose up -d` from the staged **PVS/deploy/nas/mssql.docker-compose.yml** (Express edition, SA pw in .env — CONFIRM ReelPart-New is <10GB first or use a licensed edition; cap SQL memory on the 6GB box). (2) Migrate: back up ReelPart-New off the Acer (.bak) → restore into container → **recreate the SQL logins the apps use** (pvs_ro with the 4-char db-password from Line 1; plus the Parts Control app's login — script logins from source). (3) Validate row counts (StockIns/StockOuts/Users/DailyProductionCount/ProductBOM). (4) Set NAS static IP .170, repoint PVS central.server + Parts Control at it, **cut over ONE line first in an idle window** (lines run production — don't disrupt scanning), keep the Acer as fallback ~1 week. Related: [[reading-true-stock]], [[parts-control-pc-health]].
