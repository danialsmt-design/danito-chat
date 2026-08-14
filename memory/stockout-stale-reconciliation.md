---
name: stockout-stale-reconciliation
description: "StockOuts holds huge stale stock from completed lots; a till-June B/C zero-out (3,016 reels / 17.77M pcs) was done 2026-08-12."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-13T14:19:28.442Z
---

**Finding (2026-08-12):** StockOuts.Quantity is never zeroed when a lot completes, so old issued reels keep their as-issued qty forever and inflate "remaining/at-line" stock. Snapshot before cleanup: **7,499 reels / 30,817,141 pcs** remaining across all lines, ~**92% (28.2M pcs) issued BEFORE August** (back to Feb 2026). Big chunks had **no line** (`(blank)` 3,358 reels / 13.6M; `----` 913 reels). Extends [[reading-true-stock]].

**Action taken (Danial-approved, "All 3,016 reels"):** zeroed every StockOut reel **issued ≤ June 2026 (YM<=202606), rank B or C** (A left alone), latest row per UID, Quantity>0. **3,016 rows → Quantity=0 (17,766,428 pcs).** Transaction-guarded (committed only because @@ROWCOUNT==3016). After: **4,483 reels / 13,045,045 pcs**; till-June B/C now 0.

**How it was done:** query via a line PC (Line 2) → DB `192.168.0.134` ReelPart-New, `pvs_ro` (has UPDATE on StockOuts). Rank from `PartRanks`. Date is nvarchar `dd-MM-yyyy` → YM = yyyy*100+MM.

**Second pass (same day):** cleared **B/C reels with remaining < 50 pcs** (all dates) — 58 reels / 1,445 pcs → 0. Backup `MCS/zero-BC-under50_BACKUP.csv`.

**RESTORE:** every zeroed row (StockOutID + old Quantity) is in the matching `_BACKUP.csv` (`MCS/zero-till-june-BC_BACKUP.csv`, `MCS/zero-BC-under50_BACKUP.csv`). To reverse: `UPDATE StockOuts SET Quantity=<old> WHERE ID=<StockOutID>`. Full pre-cleanup dump: `MCS/all-lines-stockout-remaining.csv`; current remainder to sort: `MCS/stockout-remainder-tosort.csv` (4,483 reels, sent to AASyaffeq Danial 60129555237 + Ashish 60102928087 — pending, WhatsApp bridge was down).

**NOT attrition:** this is a one-off stale-stock reconciliation — deliberately NOT written to `PartAttrition`/`ConsumedReels` (see [[pvs-reel-retire-rank-rule]]) so the bi-weekly Raja Rao attrition report stays production-only. Remaining candidates for a future pass: July B/C, and the A-rank stale reels (need Danial's call). This is core MCS Ai material-control work.
