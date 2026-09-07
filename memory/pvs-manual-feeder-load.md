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

**PLACEMENT COUNT IS MANDATORY (2026-09-07, `76684a2`, L1):** a pen-drive feeder list must carry the per-feeder placement count — Sony export "Mount Step" (col 4) or JUKI/Canon "QTY". The parser used to drop it, so every manual feeder decremented at 1×panels; on L1 12 of 27 feeders really placed 8–76/panel → reels ran out with PVS showing thousands, ~55k pcs false attrition in one morning. Now: the count is parsed; a file without it is REJECTED ("reject the different format from the feeder list in the Parts Control PC" — Danial) unless a supervisor forces it (then DB feeder map by machine+feeder/part). Danial says the right source is the **"com-parts list by line" from the Parts Control PC** — sample file still needed to accept that layout explicitly. The L1 pen-drive load "was a mistake": retired, DB list live.
