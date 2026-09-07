---
name: pvs-parts-requests
description: "Parts requests: PVS forecast files a request on MCS 45 min before run-out; store Terminals 1+2 show it; store keeper scans reels (= StockOut), sends the robot, or dismisses (90-min cool-down). Live 2026-09-07."
metadata:
  type: project
---

**Built + live 2026-09-07 (Danial's spec):** "PVS should instruct MCS for the parts, MCS opens a full page and requests the parts to be put on the robot"; "MCS only displays on Terminal 1 and 2, it's up to the store keeper to act"; "the storekeeper then decides to send parts or dismiss the request".

- **PVS side** (`Pvs.Core/Requests/PartsRequestPlanner.cs` pure + `Runtime/PartsRequestService.cs`): reads its OWN `/api/exhaust` (same forecast the operator sees), files one request per part when `needsRequest` and run-out ≤ `robot.requestMinutes` (45), no spare staged; closes after 2 agreeing passes or lot change; a **Dismiss** = no re-file for `robot.dismissCooldownMinutes` (90). PVS writes no stock. Chip 📦 on verify.html.
- **MCS side** (`app/Requests/*`, `wwwroot/requests.html`): tables `PartsRequests`/`PartsRequestReels` in the MCS catalog (created with `sa`, app login has no CREATE TABLE). `go.html` polls `/api/requests/count` and jumps to the board; the board beeps, shows LINE/part/run-out countdown/rack location/oldest reels, scan box (each scan = StockOut INSERT `REQ-<id>`), Send robot (dispatcher call, reason REQ-id), Dismiss. Returns to launcher after 2 min quiet.
- **Ownership unchanged:** PVS proposes, MCS issues, robot dispatcher moves, operator's feeder scan loads.
- **Lesson:** never declare an optional service (`IRobotClient?`) as a minimal-API handler parameter — it becomes an inferred BODY param and every route 500s when unregistered.

Related: [[reel-delivery-robot]], [[pvs-partsout-expected-watchdog]], [[pvs-spare-vs-loaded]], [[pvs-mcs-coordination]].
