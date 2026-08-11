---
name: pvs-manual-feeder-load
description: PVS supervisor pen-drive feeder-list override for breakdown reshuffles — loads Sony/JUKI CSVs per machine.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T15:19:17.954Z
---

When a machine breaks down mid-lot and its feeders are shared across the surviving cells, the DB ProductBOM
no longer matches the floor. The supervisor loads each running machine's **exported feeder-list CSV from a pen
drive** on the ⚙ setup page (one 📂 button per machine, badge-gated L2+). PVS then verifies against those files
instead of the DB until *Reload from DB*. Persists across restart (manual-feeders.json); audit-logged.

Key rules (see [[pvs-feeder-source-productbom]]):
- **Every row in a loaded file goes to the chosen machine** (physical location), regardless of the file's Cell#
  column. A Sony "FULL MCx" export already contains the redistributed parts from the down cell tagged with their
  original Cell# — they still land on the machine the supervisor loaded them into.
- Parser (`SonyFeederCsv.Parse(csv, machine)`) auto-detects **both** floor formats: Sony (`Cell#,[F]108 (F),Part`)
  and JUKI RS-1 (`FEEDER NO,PARTS NAME,QTY` with `F11` feeders, no Cell#).
- Single-cell Sony files are validated against the button (DeclaredCell); mixed/JUKI files skip that check.
- **Loaded total must equal the Canon BOM part count.** Verified example (L264 A-side, cell-3 down): JUKI MC1=3 +
  MC2=14 + FULL-MC4=6 = 23 = Canon BOM.
- Endpoints: GET `/api/feeders/manual/status`, POST `/api/feeders/manual` (machine+csv+badge), POST
  `/api/feeders/manual/clear`. Manual feeders carry no per-board qty, so they're untracked for the live exhaust
  forecast — the verification list + Canon-BOM total are what matter during a breakdown.
