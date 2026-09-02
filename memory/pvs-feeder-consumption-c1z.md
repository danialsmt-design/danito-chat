---
name: pvs-feeder-consumption-c1z
description: "The machine's own per-feeder pickup counts (C1Z) are the truth for feeder consumption + attrition, replacing boards×perBoard."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-09-01T13:24:39.274Z
---

Feeder draw-down and attrition should come from each machine's OWN per-feeder pickup counts (the Sony **C1Z** "Summary by Supply Location" report), NOT from `boards × placements-per-board`. Confirmed by Danial 2026-08-27.

**The rule (per feeder, per lot — C1Z counters reset when the operator resets the machine each lot, starting a fresh report page):**
- **Reel remaining = reel_start − Attempted pickups (VC).** Every attempt removes a part from the tape, so VC is what physically left the reel.
- **Parts used (placed on boards) = Successful pickups (TC).**
- **Attrition (waste) = VC − TC = Missed (MC) + Abnormal (DC) pickups.**

C1Z tags (verified by arithmetic on live L1 data): VC=attempted, TC=successful (=VC−MC−DC), MC=missed, DC=abnormal, RC=recognition, EP=parts-out stops, PR=pickup rate 1/100% (=(VC−MC−DC−RC)/VC). Parser = `Pvs.Core/Serial/SonySupplyReport.cs`; these SI-F machines send C1Z as PLAIN ASCII (manual says ZIP — they don't).

**Why:** `boards×perBoard` assumes zero pickup waste and rides on PVS's board count, which drifts when serial R0s are missed. Live proof: L1 F114 WA2-2419-000 read PVS=0 while physically 80 remained — a baseline/drift error that the machine's own pickup count would not have. See [[pvs-partsout-expected-watchdog]], [[pvs-count-two-counters-and-cap]].

**How to apply:** hybrid — keep the live per-board decrement for responsiveness, then SNAP each feeder's remaining to `start − VC-since-reel-load` on every C1Z capture (machine = truth, board count fills the gaps). Record each feeder's VC at reel-load as the baseline; a reel spanning a lot uses current VC. At parts-out, final attrition = VC−TC (true short-yield), replacing the drift-prone tracked-remainder that inflated `PartAttrition` (the 651-pc WA2-2419-000 case). Board count stays as the fallback when C1Z is unavailable (A4E00 refusal / between captures). C1Z is refused (A4E00) the same way C1M is — can be durable (M1 refused 3h on 2026-08-27), so the fallback matters.

**BREAKTHROUGH 2026-09-01 — C1Z NEEDS THE PROGRAM NAME.** The whole "C1Z is refused" saga was sending the WRONG form. On the floor, verified live on L3 M2: bare `C1Z000` → **A4E01 (unknown command)**; `C1Z000P<full program name>` (e.g. `C1Z000PL264 - A SIDE_Cell2`, program from C3P minus `.PW4`) → **lands the full per-feeder report** (9,145 bytes; 8/8 attempts while producing). The program name is MANDATORY — exactly what Danial kept saying ("add the entire programme name"). Earlier notes claiming "C1Z refused A4E00 like C1M" were from testing the bare form. The report is plain-ASCII `SD…ED…` + `Z<loc>,VC…,TC…,MC…,DC…,RC…,SC…,NC…,BC…,EP…,PR…` per feeder; PR (rate, 1/100%) reads 99.94–100% live = the accuracy band. C1M (machine-level board count) IS recognized bare (`C1M000P<name>`); C1N/C1ZC also A4E01 (unsupported). Command frame for C1Z000: `02 30 36 43315A303030 33 43 03` (STX + "06" len + "C1Z000" + "3C" cksum + ETX) — byte-identical across lines.

**Auto-capture built 2026-09-01 (all 5 lines):** `CounterReconcilerService` has a C1Z rotation timer (~45s, ONE machine per tick, no contention) that sends `C1Z000P<program name>`, self-heals a missing program via C3P, skips offline machines, polls until it lands → `RawSupplyReport` → `/api/feederstats` → machine-inv. Only lands while the machine is PRODUCING and its program is known (idle/stopped lines have empty program → nothing to capture). Every serial send is now logged (payload+hex). Console diag page: `/console.html`. NOTE this SUPERSEDES/corrects the "refused A4E00" and "board-count fallback" framing below — with the program name it is NOT refused during production. See [[pvs-component-decrement-bible]] (board-count bible is the fallback/simulation; C1Z VC/TC is the machine's own per-feeder truth).

**STATUS 2026-08-27:** SHADOW mode (`c1zShadow` config flag) DEPLOYED to all 5 lines and enabled — logs per-feeder drift (PVS per-board vs machine pickups) to `C:\PvsLineApp\supply-report\shadow-M{n}-{date}.log`, writes NO stock. Also live: `/api/feederstats` + machine-inv.html per-feeder pickup/error columns + per-lot "save finished lot" file (`lot-{lot}-M{n}.txt`, fires on lot change before the operator's reset) + retry-on-A4E00 in the reconciler + TI transaction-ID gap detector. `c1zDecrement` flag exists but the APPLY logic is NOT wired yet — that's the next phase, only after shadow validates on real production. Shadow fills when machines answer C1Z, i.e. when producing a fresh lot again (Danial: "it will answer when it start produce again").
