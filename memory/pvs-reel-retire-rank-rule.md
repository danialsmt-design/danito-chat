---
name: pvs-reel-retire-rank-rule
description: PVS auto-retires spent remnant reels by stock rank on a mid-lot reel swap; PartRanks + ConsumedReels; reversible.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-12T07:40:40.921Z
---

**Rank source:** `PartRanks` table in ReelPart-New — columns `PartNumber (varchar)`, `Rank (char)`. 155 parts: A=22, B=40, C=93. (Not in code/BOM/StockIns — it's this table.) Read via `IReelPartRepository.GetPartRankAsync`.

**Retire thresholds** (retire a reel when its remaining is below): **A → only 0 · B → < 30 · C → < 200.** Un-ranked parts are left alone.

**Auto trigger (built 2026-08-12, PVS):** when an operator swaps a reel OFF a feeder **during a running lot** (Mode-B parts change), PVS checks the OUTGOING reel's rank + remaining; if below threshold it **records it in `ConsumedReels`, then zeroes its `StockOuts.Quantity`** (via `UpdateReelQtyAsync` — the same write `SyncStockOuts` uses; zeroing is free). Records BEFORE zeroing, so if `ConsumedReels` doesn't exist the insert throws and nothing is zeroed → feature is INERT until the table exists. Gated to `SyncStockOuts=true` lines. Hook is in `SessionCoordinator.ScanReel` (capture outgoing before `_reels.Set`, retire after `MaybeFinalizeAsync`).

**Reversible:** `POST /api/reel/restore {uid, badge}` → `RestoreConsumedReelAsync` (L2+): restores the StockOut qty to `RemainingAtRetire` and sets `ConsumedReels.Restored=1`.

**Attrition:** the zeroed remainder is written-off material — accumulated per part in the `PartAttrition` table (`PartNumber`, `AttritionPcs`, `Reels`, `LastAt`) via `AddPartAttritionAsync` (+ on retire, − on restore).

**Bi-weekly report:** scheduled task `pvs-attrition-report-rajarao` (cron `0 8 1,15 * *`) sends the accumulated attrition (+ value from MCS-model-part-prices.csv) to **Raja Rao, WhatsApp 60163327003**, every 1st & 15th. Runs standalone: recovers Tailscale, queries `PartAttrition` via Line 1's PC, WhatsApps the summary. Empty until the tables exist + retires happen.

**The rest (not swapped, just sitting on the rack):** NOT automated — a list of below-threshold staged reels is sent to Ashish (WhatsApp **60102928087**) to zero manually. Generated from each line's `/api/staged` joined to `PartRanks` (see `MCS/consumed-candidates.csv`, `MCS/PartRanks.csv`). Watch out: `/api/staged` returns 0 on a transient DB read ("wider query unavailable") — retry before trusting it. See [[reading-true-stock]], [[parts-control-db-write-access]].

**LIVE (2026-08-12):** Ashish ran `deploy/ConsumedReels.sql` on dbo — `ConsumedReels` + `PartAttrition` exist and `pvs_ro` can INSERT/UPDATE both (verified via rolled-back test). Retire+attrition bundle deployed to **Line 1** (restore endpoint responds); rolling to lines 2/3/4/5 on idle (watcher). Line 3 has SyncStockOuts=false so retire is inert there. First real retire fires on the next operator parts-change of a sub-threshold reel → populates the tables → shows in the bi-weekly Raja Rao report. This is groundwork for the MCS Ai autonomous material-control agent Danial wants.
