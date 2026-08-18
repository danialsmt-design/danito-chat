---
name: pvs-system-brain
description: The connected PVS model + symptom→cause diagnostic map. Reason FROM this before acting; connect new facts back to it.
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T07:50:03.518Z
---

**Danial's feedback (2026-08-18):** "You have to build a second brain — I'm teaching you a lot and you're not putting things together and figuring out what's happening." I was capturing facts as isolated notes and re-discovering things already taught. Fix = a CONNECTED model I reason from, not scattered recall.

**The connected model lives at** `Documents/Dantec/DANOTTO/PVS-System-Brain.md` — the reel-lifecycle spine, the StockOut invariant, rank/tray handling, the consume rule, the wire signals (parts-out `R2E03`, port=cell alignment), and a **symptom→cause→check diagnostic map**. It ties together every atomic PVS note.

**How to apply — every PVS symptom:** locate it on the reel-lifecycle spine → test against the invariant (StockOut = in-feeder + standby) → run the diagnostic map to a likely cause → verify with the named check → THEN act. **Connect each new fact back into the model** rather than filing a standalone note. Don't make Danial re-teach — if he's explained a mechanism, it's in the brain; consult it first.

Worked example of the payoff: "L4 M1 produces but 0 parts-out" — the map says check port=cell alignment FIRST (before parser); that was it (cross-wired), after a long detour that the model would have short-cut. See [[pvs-port-cell-alignment]], [[pvs-partsout-expected-watchdog]], [[pvs-spare-vs-loaded]], [[pvs-reel-retire-rank-rule]], [[pvs-rules-bible]], [[second-brain-vault]].
