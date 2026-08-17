---
name: pvs-spare-vs-loaded
description: "Canonical rule — a reel on a feeder is production, never a spare. Spare/standby/forecast count = StockOut reels NOT on a feeder."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-17T08:07:44.296Z
---

**Danial's rule (2026-08-17), canonical — do not violate:** A reel physically **loaded on a feeder is PRODUCTION, never a spare.** The **spare / standby / "next-to-run-out spare" count** for a part = **only its StockOut reels that are NOT on any feeder** (issued to the line, non-zero, and NOT in that line's `inventory.json` loaded set). Full stop.

**Why:** conflating the two double-counts and misleads the floor. A loaded reel must never appear in a spare/standby list or pad an exhaust-forecast "spare ×N" number.

**How to apply — every time:**
- Any "spare / at-line / standby" figure = `ReelsAtLine (StockOuts issued to line, latest row per UID, Qty>0)` **minus** `loaded UIDs (inventory.json line.Reels.All())`. The app already computes it this way; keep it that way.
- The exhaust card's per-part `spareReels` and the standby-check "believed" list both already exclude loaded reels — never re-add them.
- When a spare count looks **too high**, the cause is **stale StockOut rows** (reels pulled off but never zeroed) — they are "not on a feeder" so they wrongly pad the count. Fix = **zero the stale not-in-feeder reels**, NOT recount loaded reels.
- Retire threshold C<200 (and B<30, A=0) is **swap-triggered only** (outgoing remnant on a physical reel change → ConsumedReels), NOT a standing sweep. See [[pvs-reel-retire-rank-rule]].
- Governed by the [[pvs-rules-bible]]; also relevant to MCS Ai stock-accuracy work ([[pvs-mcs-coordination]], [[stockout-stale-reconciliation]]).
