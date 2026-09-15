---
name: pvs-attrition-report
description: The 2 % attrition rule — PVS remaining at a genuine parts-out ÷ reel start qty; over 2 % escalates to Raja Rao by WhatsApp at lot end; C1Z is the throw alarm, never the count.
metadata:
  type: project
---

Danial's accuracy rule (2026-09-14): "the component shortage during parts exhaust compared with PVS creates an attrition report, should be less than 2 %, anything more highlight to Mr Raja." A parts-out E03 alone is a FALSE CALL (the machine sends E03 at lot start for an unthreaded feeder on a full reel — every line read 95–100 % on 2026-09-15 and one false WhatsApp reached Raja Rao). Danial: "E03 is correct only if the operator scanned the old reel and the new reel ID." So the sample is taken only when the parts-out change completes with old reel + different new reel scanned. At that confirmed exhaust the reel is empty, so whatever PVS still shows on that reel is the shortage. percent = shortage ÷ start qty. > 2 % = OVER (red on verify.html + report.html + /api/attrition); OVER with ≥ 20 panels on the reel = ONE WhatsApp per lot to Raja Rao (60163327003) at lot end. Built + deployed to all 5 lines 2026-09-14 (`d51f3c4`), unproven until the first parts-out/lot end.

**Why:** C1Z is intermittent so per-feeder machine pickups cannot be the count; the parts-out is the only exact event. The report tells whether boards × placements (calibrated to lot size / lot completed, written to StockOut) is holding, and a reel far over 2 % usually means a wrong reel on the feeder, a short reel from the store, or a throwing feeder.

**How to apply:** never propose C1Z for counting again; C1Z = high-throw feeder alarm only ([[pvs-feeder-consumption-c1z]]). Don't escalate short runs (< 20 panels). Keep the report read-only on counts ([[pvs-component-decrement-bible]]). Contact numbers in [[frequent-contacts]].
