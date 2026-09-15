---
name: pvs-e03-needs-both-scans
description: Day-one PVS rule — a Sony parts-out (E03) is a fact only after the operator scans the OLD reel and a DIFFERENT new reel; a raw E03 is a false call. Never hook anything to the raw frame.
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 0f34bd21-679d-4bff-bbbb-863d5c757f47
  modified: 2026-09-15T09:20:48.863Z
---

A parts-out E03 from the machine is NOT a consumed reel. It is confirmed only when the operator scans the old reel (the one PVS tracks) and a different new reel onto that feeder, i.e. the parts-change session COMPLETES. Anything else (E03 at lot start on an unthreaded feeder, a skip, the same reel re-seated) is a false call. Danial: "parts-out E03 are correct only if the operator scanned the old reel and the new reel ID, otherwise it's a false call — this is basic rule from day one."

**Why:** On 2026-09-15 I sampled the attrition report on the raw E03; every line read 95–100 % shortage at 0–10 boards and one false WhatsApp reached Raja Rao. The retire path (`RetireOutgoingReelAsync`) already obeyed the rule by running only from the completed change.

**How to apply:** any new logic that means "the reel is finished" (attrition, exhaust calibration, retire, StockOut zero, alarms to people) hangs off the parts-change COMMIT in `MaybeFinalizeAsync` with `FromPartsOut`, `OldReelUid` = tracked UID, `NewReelUid` different — never off `OnPartsOut`. `OnPartsOut` only queues the pending change. See [[pvs-attrition-report]], [[pvs-component-decrement-bible]].
