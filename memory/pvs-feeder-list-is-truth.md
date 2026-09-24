---
name: pvs-feeder-list-is-truth
description: The feeder list placement count is trusted 100% (the product would not qualify otherwise); PVS never learns or adjusts a rate. Reel balance = loaded qty − list × line panels since load; a reel that empties early/late means the reel/StockOut quantity was wrong, never the list.
metadata:
  type: feedback
---

Danial 2026-09-24: "the feeder list can't be wrong or the product will not qualify" — "the feeder list must be trusted 100%". "If the reel lasts longer, the StockOut count is wrong."

**Why:** I had built a learned per-part rate from confirmed exhausts and was about to apply it to the count-down. That would have hidden the real faults: a reel issued with the wrong quantity, a mislabelled reel, throws. Example: a 3,000-piece reel at 16/panel lasted 212 panels → it held 3,392; the StockOut record was 392 short.

**How to apply:** the count-down and the reconcile use the feeder-list count only (`BalanceReconciler.RateFor` returns the list; the real rate from an exhaust is report-only). At exhaust, reel count = list × panels since load; the difference to the loaded qty is a REEL/StockOut error (negative = reel held more than recorded) and belongs in the attrition report, not in a rate. Never propose learning/adapting placement counts again. See [[pvs-attrition-report]], [[pvs-component-decrement-bible]].
