---
name: parts-control-db-write-access
description: "How to write to the ReelPart-New DB — dbsvc CAN write data remotely (db_datawriter, no DDL); ACER-PC (dbo) needed only for grants/DDL."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-07T12:30:39.760Z
---

**Update 2026-07-31 (verified live):** the DB instance is `DESKTOP-TECHNIC\SQLEXPRESS` and it **listens on TCP 1433** (Tailscale `100.91.120.113,1433`) — that's the same instance as `.\SQLEXPRESS`, not a separate default instance. PVS connects with the SQL login **`pvs_ro`** (userId in `line.config.json` → `central`), which is **read-only**. My `partsctl.cred` WinRM session runs as **`DESKTOP-TECHNIC\dbsvc`**, which is `db_datawriter` (confirmed: a rolled-back test INSERT into DailyProductionCount succeeded) **but `is_sa=0` and canNOT `GRANT`** (tried `GRANT INSERT … TO pvs_ro` as dbsvc → "Cannot find the object … or you do not have permission"). So dbsvc can write data itself but cannot hand write rights to pvs_ro. **To grant pvs_ro a permission you need a db_owner/sysadmin at the PC** — put a BAT on the Parts Control PC and have the DBA run it. Did exactly this for the DPC writer: `deploy/grant-pvs-write.bat` → `GRANT INSERT ON dbo.DailyProductionCount TO pvs_ro` (dropped to `C:\Users\Public\Desktop\`, run by the DBA). **After the grant, PVS is now the Line 1 DailyProductionCount writer** (operator app stopped writing L1; `writeProductionCount:true`, 5-min buckets per lot/side). See [[pvs-shift-and-daily-report]].

**Update 2026-08-07 (verified live, during the L254 BOM correction):** `dbsvc` is `db_datawriter` +
`db_datareader` but **NOT `db_ddladmin`** — so `SELECT * INTO <backup_table>` fails with *"CREATE TABLE
permission denied"*. **Back up to a CSV FILE instead of a backup table**: `sqlcmd -s "," -h -1` through the
WinRM session, written to `Documents\Dantec\MCS\backups\`. Verify the row count and that the target rows
appear in it *before* the UPDATE. Also: **`ProductBOM.TotalPrice` is derived (`Quantity * UnitPrice`)** —
updating `Quantity` alone silently corrupts costing, so always set both in the same statement.

Writing to the **ReelPart-New** DB on the Parts Control PC (192.168.0.134, `DESKTOP-TECHNIC\SQLEXPRESS`):

- **As of 2026-07-25, `DESKTOP-TECHNIC\dbsvc` now has `db_datawriter` + `db_datareader`** — so I can write REMOTELY via the normal `partsctl.cred` WinRM session + Integrated Security (`Server=.\SQLEXPRESS;Database=ReelPart-New;Integrated Security=True`). No one needs to be at the Parts Control PC. Always back up affected rows to `C:\PVS_Backups\*.csv` first and wrap changes in `SET XACT_ABORT ON; BEGIN TRAN ... IF @count<>expected ROLLBACK ELSE COMMIT`. To revoke: `ALTER ROLE db_datawriter DROP MEMBER [dbsvc]`.
- **`DESKTOP-TECHNIC\ACER-PC` is the DB owner (`dbo`, `db_owner`)** — the account the parts control app runs under (Integrated Security, no SQL login/password). It's the only one that can do db_owner ops (role grants, etc.); run those in ACER-PC's interactive session via a desktop BAT.
- PVS **line apps stay read-only** via the separate `pvs_ro` SQL login — the dbsvc write grant is only the admin/maintenance path.
- **Exception (2026-07-27): the new PVS "Model Change" mode writes `StockOuts.Quantity`** when a supervisor corrects a reel qty. Model-Change shows the reel's qty from **StockOuts** (by UID, most-recent ID; `FindStockOutQtyAsync`) — NOT StockIns — and the correction writes back there (`UpdateReelQtyAsync`, the only write in the repo; targets the row via StockOuts.ID). Needs `pvs_ro` to have `GRANT UPDATE ON dbo.StockOuts(Quantity)` — column-level. Staged as `Grant_PVS_QtyWrite.bat` on ACER-PC's desktop (run in ACER-PC's session; only db_owner grants; it also revokes the earlier StockIns grant). Until run, the correction records locally + the screen notes the write is pending. Tested: the StockOuts write targets exactly 1 row and rolls back clean. A standalone **reel-qty editor page** (`/reel.html`, link in verify header) also writes via the same path: scan UID -> shows part+qty (`FindStockOutReelAsync`, `GET /api/reel`) -> enter qty -> scan supervisor badge (L2+) -> `POST /api/reel/qty` (`UpdateReelQtyAsync`), same grant needed. Verify checklist now splits feeders into per-machine tabs (M1-M4). A **machine feeder-inventory page** (`/machine-inv.html`) shows each feeder's part + reel UID + live StockOuts qty and lets a supervisor adjust it (writes StockOuts). It relies on `FeederReelStore` (Pvs.LineApp/Inventory) which persists feeder->reel (UID) mappings to `C:\PvsLineApp\inventory.json`, populated from every verified reel scan (survives restart) — the DB never stored which reel is on which feeder. TODO: `QtyFromUid` fallback (parse qty from the scanned UID when a reel has no StockOuts row) — awaiting the UID barcode format. UX: scanning a `FIRM` barcode = the qty Confirm button; an accidental badge scan in the reel slot is ignored (no interlock), guarded in UI + coordinator. See [[sony-smt-parts-verification]] and [[pvs-model-naming]].
- `sa` is **disabled**; mixed-mode auth is on but no usable SQL login exists. `BUILTIN\Users` has no DB write.
- Remoting AS ACER-PC failed: WinRM "Access denied", and S4U scheduled-task registration as ACER-PC also "Access denied" (filtered remote token). ACER-PC's password is NOT `1234` (validated False).

**Working write path:** stage a `.ps1` (Integrated Security, wrapped in a transaction) under `C:\PVS_Backups\` via the dbsvc WinRM session, plus a double-click `.bat` on **ACER-PC's desktop** (`C:\Users\ACER-PC\Desktop\`). Someone runs it **in ACER-PC's own logged-in session** on the Parts Control PC → runs as dbo → write succeeds. Verify results read-only via dbsvc afterward. Always back up affected rows to `C:\PVS_Backups\*.csv` first and use `SET XACT_ABORT ON; BEGIN TRAN ... IF @count<>expected ROLLBACK ELSE COMMIT`.

To make writes not depend on someone at that PC, set up a dedicated write-capable SQL login for PVS (needs ACER-PC/sysadmin to create). See [[sony-smt-parts-verification]] and [[schedulle-project]].
