---
name: pvs-count-reconcile-timing
description: "Sony board cycle is ~40s; the safe moment to sample/reconcile the machine count is right after a board completes, in that gap — not on a blind timer."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T04:03:03.837Z
---

Danial (floor knowledge, 2026-08-11): on the Sony lines a board completes roughly every **40 seconds**, and there's a **~40s idle gap after each board-complete before the next board starts**. The **safe moment to read/reconcile the machine count is right after a board-complete, inside that gap** — reading mid-cycle risks losing/mis-counting a board.

**Why:** PVS's *live* lot count is already event-driven — `_m4PanelsTotal++` fires exactly once per board-complete (R0) signal, so the live tally itself doesn't lose count from timing. The risk is the periodic **CounterReconcilerService**, which reads the machine's C1M counter on a blind timer (`config.ReconcileMinutes`) and can sample mid-cycle.

**How to apply:** trigger the reconcile's count-sample **off the board-complete event** (delayed into the ~40s post-board window) rather than a fixed interval, so the C1M read is always taken when the counter is stable between boards. Pairs with the reset-guard in [[pvs-count-two-counters-and-cap]] (auto-adopt refuses a backwards/reset value). Not yet implemented — queued with the other count-integrity refinements.
