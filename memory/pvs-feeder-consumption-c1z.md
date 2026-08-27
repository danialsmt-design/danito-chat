---
name: pvs-feeder-consumption-c1z
description: "The machine's own per-feeder pickup counts (C1Z) are the truth for feeder consumption + attrition, replacing boards×perBoard."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-27T14:21:41.773Z
---

Feeder draw-down and attrition should come from each machine's OWN per-feeder pickup counts (the Sony **C1Z** "Summary by Supply Location" report), NOT from `boards × placements-per-board`. Confirmed by Danial 2026-08-27.

**The rule (per feeder, per lot — C1Z counters reset when the operator resets the machine each lot, starting a fresh report page):**
- **Reel remaining = reel_start − Attempted pickups (VC).** Every attempt removes a part from the tape, so VC is what physically left the reel.
- **Parts used (placed on boards) = Successful pickups (TC).**
- **Attrition (waste) = VC − TC = Missed (MC) + Abnormal (DC) pickups.**

C1Z tags (verified by arithmetic on live L1 data): VC=attempted, TC=successful (=VC−MC−DC), MC=missed, DC=abnormal, RC=recognition, EP=parts-out stops, PR=pickup rate 1/100% (=(VC−MC−DC−RC)/VC). Parser = `Pvs.Core/Serial/SonySupplyReport.cs`; these SI-F machines send C1Z as PLAIN ASCII (manual says ZIP — they don't).

**Why:** `boards×perBoard` assumes zero pickup waste and rides on PVS's board count, which drifts when serial R0s are missed. Live proof: L1 F114 WA2-2419-000 read PVS=0 while physically 80 remained — a baseline/drift error that the machine's own pickup count would not have. See [[pvs-partsout-expected-watchdog]], [[pvs-count-two-counters-and-cap]].

**How to apply:** hybrid — keep the live per-board decrement for responsiveness, then SNAP each feeder's remaining to `start − VC-since-reel-load` on every C1Z capture (machine = truth, board count fills the gaps). Record each feeder's VC at reel-load as the baseline; a reel spanning a lot uses current VC. At parts-out, final attrition = VC−TC (true short-yield), replacing the drift-prone tracked-remainder that inflated `PartAttrition` (the 651-pc WA2-2419-000 case). Board count stays as the fallback when C1Z is unavailable (A4E00 refusal / between captures). C1Z is refused (A4E00) the same way C1M is — can be durable (M1 refused 3h on 2026-08-27), so the fallback matters.
