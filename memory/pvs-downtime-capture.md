---
name: pvs-downtime-capture
description: "PVS downtime capture — auto per-cell parts-exhaust recovery + operator reason pop-up by line. BUILT 2026-08-18, tests pass, NOT deployed (deploy 2026-08-19)."
metadata:
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T09:08:41.166Z
---

**BUILT 2026-08-18 in the PVS source, NOT yet deployed (Danial: deploy tomorrow / 2026-08-19).** Builds clean, all 365 Core tests pass (12 new).

**Two halves (design settled with Danial over this session):**
1. **AUTOMATIC per-cell parts-exhaust recovery** — every parts-out is timed to that CELL producing again; per-cell tally = # exhausts + recovery time (total/avg/worst). No operator action (PVS sees parts-out + board-complete on the wire).
2. **MANUAL operator reason stops BY LINE** — pop-up on verify.html with 5 buttons: **Machine problem / Waiting part / No air / Rest time / Scheduled stop.** Recording **starts ONLY when the operator taps a reason** (clock from the tap, not the detected stop); **ends when the line produces again**, or at **shift end** if the line never runs again. Re-tapping a different reason re-classifies. Report tallies total time **under each title**.

**Files:**
- `Pvs.Core/Runtime/ReasonStopLog.cs` + `CellRecoveryLog.cs` (pure, unit-tested; `tests/Pvs.Core.Tests/StopCaptureTests.cs`).
- `Pvs.LineApp/Runtime/StopTrackingService.cs` — subscribes each channel's `PartsOutDetected` + `BoardCompleted`, persists `stops.json` (per-day, restart-safe, resets at day boundary), closes open stop at shift rollover. Part tag via new `SessionCoordinator.ExpectedPartAt(m,f)`.
- Wired in `LineService` (property `StopTracking`, start + dispose). Config `LineConfig.StopDetectSeconds` (default 90 = ~2 missed cycles; the "stopped producing" prompt threshold).
- Endpoints: `GET /api/stop/state` (open reason + elapsed, `stopped`/`producing` flags, open cell recoveries), `POST /api/stop/reason {reason}`. `/api/report/daily` gains `stopCapture` (byReason + byCell + stops + recoveries).
- `verify.html`: `#stopPop` overlay (5 bilingual EN/MY buttons), `#dtChip` recording chip (⏺ reason · mm:ss) + `#dtBtn` "Line stop" in the Controls card. Auto-prompts when `stopped && !dismissed && no check-overlay`; clock still starts on tap. Verified in a static preview (chip formats, pop-up renders, no JS errors).

**NOT in this build (still to do / discuss):**
- **Break windows** — Danial will feed break times; then a down span in a break tags **Break** + is excluded from downtime, and **break-aware recovery** subtracts overlapping break minutes from a cell's parts-exhaust recovery. See [[pvs-shift-and-daily-report]] downtime + [[pvs-board-flow]].
- **"Waiting boards from Line X"** reason (cross-line magazine starvation) — depends on the board-input tracking ([[pvs-board-flow]] / [[pvs-bare-board-pack-capture]]), the next build chunk.
- `report.html` UI sections for the new rollups (API is live; page not yet updated).

Connects: [[pvs-shift-and-daily-report]], [[pvs-board-flow]], [[pvs-partsout-expected-watchdog]], [[pvs-system-brain]].
