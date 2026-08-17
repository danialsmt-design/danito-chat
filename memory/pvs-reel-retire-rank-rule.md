---
name: pvs-reel-retire-rank-rule
description: PVS auto-retires spent remnant reels by stock rank on a mid-lot reel swap; PartRanks + ConsumedReels; reversible.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-17T15:07:18.071Z
---

**Rank source:** `PartRanks` table in ReelPart-New — columns `PartNumber (varchar)`, `Rank (char)`. 155 parts: A=22, B=40, C=93. (Not in code/BOM/StockIns — it's this table.) Read via `IReelPartRepository.GetPartRankAsync`.

**⚠ CHANGED 2026-08-17 — the rank threshold was REMOVED as the retire gate.** Danial's governing model (see [[pvs-spare-vs-loaded]] and the [[pvs-rules-bible]]): **StockOut = reels in feeders + standby reels at the line; a CONSUMED reel leaves StockOut for `ConsumedReels`.** A **parts-change swap = the machine ran that feeder OUT = the outgoing reel is consumed (empty)** — the drift-prone tracked remaining doesn't matter. So the retire now fires **UNCONDITIONALLY** on a parts-change swap, for **any rank, any remaining**. The old thresholds (A:0 / B:<30 / C:<200) both stranded drift-inflated empties (→ StockOut bloat) and over-retired usable partials — that's why they're gone.

**Auto trigger (built 2026-08-12, gate removed 2026-08-17):** when an operator swaps a reel OFF a feeder during a running lot (Mode-B parts change), PVS **records the OUTGOING reel in `ConsumedReels`, then zeroes its `StockOuts.Quantity`** (via `UpdateReelQtyAsync`). Records BEFORE zeroing, so if `ConsumedReels` doesn't exist the insert throws and nothing is zeroed → INERT until the table exists. Gated to `SyncStockOuts=true` lines. Hook is `RetireOutgoingReelAsync` in `SessionCoordinator.ScanReel` (capture outgoing before `_reels.Set`, retire after `MaybeFinalizeAsync`). `RetireThreshold` method deleted. Rank is still read but only stamped on the ConsumedReel for the restore record. **Deployed to all 5 lines 2026-08-17** (L2/3/5 WinRM, L1/4 SMB+RPC since their WinRM is down).

**Reversible:** `POST /api/reel/restore {uid, badge}` → `RestoreConsumedReelAsync` (L2+): restores the StockOut qty to `RemainingAtRetire` and sets `ConsumedReels.Restored=1`.

**Attrition:** the zeroed remainder is written-off material — accumulated per part in the `PartAttrition` table (`PartNumber`, `AttritionPcs`, `Reels`, `LastAt`) via `AddPartAttritionAsync` (+ on retire, − on restore).

**Bi-weekly report:** scheduled task `pvs-attrition-report-rajarao` (cron `0 8 1,15 * *`) sends the accumulated attrition (+ value from MCS-model-part-prices.csv) to **Raja Rao, WhatsApp 60163327003**, every 1st & 15th. Runs standalone: recovers Tailscale, queries `PartAttrition` via Line 1's PC, WhatsApps the summary. Empty until the tables exist + retires happen.

**The rest (not swapped, just sitting on the rack):** NOT automated — a list of below-threshold staged reels is sent to Ashish (WhatsApp **60102928087**) to zero manually. Generated from each line's `/api/staged` joined to `PartRanks` (see `MCS/consumed-candidates.csv`, `MCS/PartRanks.csv`). Watch out: `/api/staged` returns 0 on a transient DB read ("wider query unavailable") — retry before trusting it. See [[reading-true-stock]], [[parts-control-db-write-access]].

**LIVE (2026-08-12):** Ashish ran `deploy/ConsumedReels.sql` on dbo — `ConsumedReels` + `PartAttrition` exist and `pvs_ro` can INSERT/UPDATE both (verified via rolled-back test). Retire+attrition bundle deployed to **Line 1** (restore endpoint responds); rolling to lines 2/3/4/5 on idle (watcher). Line 3 has SyncStockOuts=false so retire is inert there. First real retire fires on the next operator parts-change of a sub-threshold reel → populates the tables → shows in the bi-weekly Raja Rao report. This is groundwork for the MCS Ai autonomous material-control agent Danial wants.
