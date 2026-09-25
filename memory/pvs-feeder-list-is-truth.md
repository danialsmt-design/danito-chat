---
name: pvs-feeder-list-is-truth
description: The feeder list placement count is trusted 100% (the product would not qualify otherwise); PVS never learns or adjusts a rate. Reel balance = loaded qty − list × line panels since load; a reel that empties early/late means the reel/StockOut quantity was wrong, never the list.
metadata:
  type: feedback
---

Danial 2026-09-24: "the feeder list can't be wrong or the product will not qualify" — "the feeder list must be trusted 100%". "If the reel lasts longer, the StockOut count is wrong."

**Why:** I had built a learned per-part rate from confirmed exhausts and was about to apply it to the count-down. That would have hidden the real faults: a reel issued with the wrong quantity, a mislabelled reel, throws. Example: a 3,000-piece reel at 16/panel lasted 212 panels → it held 3,392; the StockOut record was 392 short.

**How to apply:** the count-down and the reconcile use the feeder-list count only (`BalanceReconciler.RateFor` returns the list; the real rate from an exhaust is report-only). At exhaust, reel count = list × panels since load; the difference to the loaded qty is a REEL/StockOut error (negative = reel held more than recorded) and belongs in the attrition report, not in a rate. Never propose learning/adapting placement counts again. See [[pvs-attrition-report]], [[pvs-component-decrement-bible]].

**2026-09-25 — the FEEDER MASTER is the list.** Per line, per model+side: MC1..MC4 rows {feeder, part, shots/board} + boards/panel, edited on `master.html` (supervisor badge, versioned, history). It is the ONLY source the count-down/forecast/checklist read; the DB feeder map and pen-drive files only FILL a block (import, unreviewed until saved). Shots are per board, whole numbers; per panel = × boards/panel. Never let another source feed the count-down again.

**2026-09-25 (later) — enforced: master or nothing (`1607026`, all 5 lines).** No block at start-up = nothing tracked (no cached-list fallback) until the live read fills it; a pen-drive CSV and the DB map are IMPORTS into the block (badge, must divide to whole shots/board), never a live source. Health signal `feederMaster` says down/warn/ok. Danial: "NO MORE EXCUSES — make sure all countdown are from the Feeder Master."

**2026-09-25 11:05 — DB import disabled (`55db44e`, all 5 lines).** Danial: "disable the DB import and refresh from DB." The DB feeder map (ProductBOM) never fills or refreshes a block; a block exists only from the pen-drive CSV import (per machine, master.html or ⚙) or hand entry. A model with no block tracks nothing. `DbImportEnabled` const in SessionCoordinator.
