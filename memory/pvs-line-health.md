---
name: pvs-line-health
description: "PVS line-health rollup — one /api/health verdict from 6 signals + a badge on the operator screen; built 2026-08-18, NOT deployed."
metadata:
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T10:32:36.037Z
---

**Line-health rollup — BUILT in source 2026-08-18, builds + 369 tests pass, NOT deployed.** One per-line verdict = the **worst of 6 signals**, each OK / WARN / DOWN (idle/na ignored):

1. **Serial link** — per machine: port open + IsOnline + last-msg age (>600s = warn). Down if any non-skipped machine's port is shut or not answering.
2. **Producing** — from `StopTrackingService.HealthInputs()`: producing = board within `StopDetectSeconds`; not-run-today = idle (not a fault); stopped = warn.
3. **Program/model** — machines-disagree OR pin `ModelVerify=="mismatch"` = down; `unverified` = warn.
4. **Check currency** — a shift check completed THIS shift (`LastShiftScan.ShiftKey == CurrentShiftKey`).
5. **DB link** — NEW `DbHealthService` heartbeat: `IReelPartRepository.PingAsync` (`SELECT 1`) every 45s; ok/down/warn; n/a when no SQL login. This is the only genuinely new signal — the rest of the app swallows DB errors silently.
6. **Downtime load** — `DowntimeService` up/down + minutes today.

**Endpoint:** `GET /api/health` → `{ status, signals{ serial, producing, program, check, db, downtime } }` (Program.cs, after `/api/serial/test`). Consolidates existing live signals; only DbHealth is new state. **UI:** a health **pill** in the `verify.html` header (green/amber/red "Line OK/WARN/DOWN") that taps to a popover listing the 6 signals — polls `/api/health` every 4s. Placed on the OPERATOR screen (Danial's call). Files: `Pvs.LineApp/Runtime/DbHealthService.cs`, `PingAsync` in `Pvs.Data`, `/api/health` in Program.cs, header pill + `#healthPop` in verify.html.

Signal inventory (what already existed vs new) was mapped by a read-only Explore subagent. Connects: [[pvs-downtime-capture]], [[pvs-partsout-expected-watchdog]], [[pvs-port-cell-alignment]], [[pvs-connection-guardian]], [[pvs-system-brain]].
