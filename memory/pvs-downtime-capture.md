---
name: pvs-downtime-capture
description: "PVS downtime capture — auto per-cell parts-exhaust recovery + operator reason pop-up by line; shift/break/coverage model; built 2026-08-18, NOT deployed."
metadata:
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T10:14:33.276Z
---

**Downtime capture feature — BUILT in source 2026-08-18, tests pass (365), NOT deployed (Danial: "code but wait deployment till tomorrow").** Two halves:

1. **AUTOMATIC per-cell parts-exhaust recovery** (`Pvs.Core/Runtime/CellRecoveryLog.cs`): every parts-out → clock to that cell producing again; per-cell rollup = exhaust count + recovery time (total/avg/worst), tagged with the part# (`SessionCoordinator.ExpectedPartAt`). No operator action — PVS already sees parts-out + board-complete on the wire.
2. **MANUAL reason stops BY LINE** (`Pvs.Core/Runtime/ReasonStopLog.cs`): buttons **machine / waiting_part / no_air / rest / scheduled**. Recording starts ONLY on the operator tap (not the detected stop); ends when the line produces again, or at shift end if it never does. Re-tap re-classifies.

**Wiring:** `Pvs.LineApp/Runtime/StopTrackingService.cs` (subscribes each channel's PartsOutDetected + BoardCompleted, persists `stops.json` per day, closes open stops on shift rollover). Endpoints `GET /api/stop/state`, `POST /api/stop/reason {reason}`; daily report gains `stopCapture` (byReason tallies + per-cell recovery). verify.html: reason pop-up `#stopPop` + recording chip `#dtChip` + "⏱ Line stop" button; auto-prompts when `stopped` (no board for `StopDetectSeconds`, default 90) and no check overlay is up. Config knob `LineConfig.StopDetectSeconds`.

**Board-input count = FIRST machine (Line 3 = M2), NOT M4.** M4 = output/lot count. See [[pvs-board-flow]].

**SHIFT / BREAK / STAFFING MODEL (Danial, 2026-08-18) — governs how breaks affect downtime:**
- **Day shift 07:30–19:30 is the normal run. NIGHT shift (19:30–07:30) is ON-DEMAND only** — high demand or breakdown recovery. Not a fixed daily shift; don't hardcode fixed night breaks.
- **Lunch is STAGGERED, two 45-min slots: 12:00–12:45 and 12:45–13:30.** Afternoon break 15:30–15:45.
- **Operators cover for each other** — when behind schedule, one operator covers **2 lines** while the other breaks, so **a line often keeps RUNNING through lunch.** A break window is therefore NOT guaranteed planned-downtime.
- **CONSEQUENCE for downtime:** trust the PRODUCTION SIGNAL (board-completes = line running), not the clock. Line producing through a break = covered, no downtime (automatic). Break windows are only a HINT: a stop inside a break slot defaults the suggested reason to **Rest time** (operator confirms/changes). **No auto-exclusion, no silent break subtraction** — idle-during-break is COUNTED and tagged Rest time (visible under its own title; net it out in a report view if wanted). `BreakWindows.cs` pure helper exists for the hint.

Connects: [[pvs-board-flow]], [[pvs-partsout-expected-watchdog]], [[pvs-system-brain]], [[pvs-shift-and-daily-report]], [[pvs-rules-bible]].
