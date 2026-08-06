---
name: parts-control-pc-health
description: "The Parts Control PC is a 4GB Acer desktop with C: at 0.1% free and ReelPart-New unbacked-up since 14 May 2026."
metadata: 
  node_type: memory
  type: project
  originSessionId: 5515b171-01eb-4dc7-b525-d548afcd29d2
  modified: 2026-07-30T01:18:53.386Z
---

Health audit of **DESKTOP-TECHNICAL** (the Parts Control PC) on 2026-07-30. It is an
**Acer Aspire XC-885**: i3-8100, **3.88 GB RAM**, one 240 GB Kingston SA400S37240G SSD (Healthy),
Windows 11 **Home** Single Language build 26200.

Two findings that matter:

1. **C: had 0.18 GB free of 112.3 GB (0.1%).** `ReelPart-New.mdf` lives on that very drive
   (`C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\DATA\`). The DB itself is small
   (72 MB mdf + 8 MB log) — **Windows** is what filled the disk, not the data. Biggest safe wins:
   `C:\Windows\SoftwareDistribution\Download` (5.87 GB) and `C:\$GetCurrent` (3.81 GB) ≈ 9.7 GB.
   `WinSxS` is 13.89 GB (shrinkable with DISM). **D: has 71 GB free and is nearly empty** — the DB
   files and/or backups belong there.
2. **ReelPart-New had no backup since 2026-05-14** — ✅ **FIXED 2026-07-30**, see below.

**Nightly backup — now in place (2026-07-30):**
- Scheduled task **`ReelPart Nightly Backup`**, daily **02:00**, runs script `D:\SQLBackups\backup-reelpart.ps1`.
- Backs up `ReelPart-New`, `ReelPart`, `ReelPart-Test` with `INIT, CHECKSUM`, then proves each one with
  `RESTORE VERIFYONLY ... WITH CHECKSUM`. Logs to `D:\SQLBackups\backup-log.txt`. **14-day retention.**
  First run: ReelPart-New 15 MB, ReelPart 4 MB, ReelPart-Test 6 MB, all VERIFIED.
- No `WITH COMPRESSION` — SQL Express does not support backup compression.

**Why it runs as `DESKTOP-TECHNIC\ACER-PC`:** only three logins exist on SQLEXPRESS — `sa`
(**disabled**), `BUILTIN\Users`, and `DESKTOP-TECHNIC\dbsvc`. **`sa` is the only sysadmin**, so there is
no usable sysadmin. `dbsvc` is only datareader+datawriter and `HAS_PERMS_BY_NAME(... 'BACKUP DATABASE')`
returns **0** — it *cannot* back up. `ACER-PC` **owns** ReelPart-New, so it maps to `dbo` and can.
The task therefore uses `-UserId 'DESKTOP-TECHNIC\ACER-PC' -LogonType Interactive` (no stored password).
`NT SERVICE\MSSQL$SQLEXPRESS` was granted modify on `D:\SQLBackups` — **the SQL service writes the .bak,
not the caller**, so that grant is mandatory.

**Two live caveats:**
- *Interactive* logon means the task **only fires while ACER-PC is logged on**. It is permanently logged
  in on the shop floor, but a logout or an unattended reboot silently stops backups. Check the log date.
- **C: and D: are two partitions of the same physical Kingston SSD.** The backups protect against
  corruption and accidental deletion, **not against disk failure.** An off-machine copy is still needed.

Also on the box: a **second SQL instance `MSSQL$BARTENDER`** (BarTender label software,
`Datastore_BarTender`) competing for the same 4 GB, plus IIS on port 80, Visual Studio (`devenv`),
and folders `C:\PvsValidate` and `C:\PvsDbCheck`.

RAM sits at ~87% of 3.88 GB with ~900 MB of pagefile in use — this box is memory-starved, which is
why remote scans crawl. Both `sqlservr` processes were only using 21 MB and 35 MB, i.e. squeezed.

This strengthens the case for [[parts-db-server-migration]]. Until that happens, the disk and the
backup gap are the two things that could actually cause an outage or data loss.

Related: [[remote-access-from-home]], [[parts-control-db-write-access]]
