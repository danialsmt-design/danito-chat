---
name: pvs-deploy-when-line-idle
description: "How to detect a PVS line is stopped (board rate, not online flag) and the deploy-when-idle watcher."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T15:19:30.200Z
---

Never deploy to a producing line — the deploy guard stops the app + swaps DLLs + restarts (~30s of verify-screen
downtime). Roll out only to idle lines.

**Detecting "stopped" remotely is subtle:** a Sony machine keeps answering PVS's serial polls with `A4E00`
(negative-ack heartbeat) even when parked, so `machine.online` stays `true` and `lastAt` keeps advancing at a
stop. The trustworthy "not producing" signal is **`boardsPerHour` going to 0/null across all online machines**
(boards stop completing). Confirm it over several consecutive polls — a jam-clear reads as zero-rate too but is
usually shorter.

`deploy/deploy-when-idle.ps1` arms this: polls each pending line's `/api/status` every ~50s, needs 5 consecutive
zero-rate reads (~4 min), refuses to fire while an operator is mid-scan (activeMode not None/ShiftChange), then
runs `deploy-pvs.ps1` (backup + health-check + auto-rollback) and drops the line. Staged bundle lives in
`deploy/_pending`; log at `deploy/_pending/deploy-when-idle.log`.

Line PC Tailscale IPs: L1 100.69.81.105, L2 100.94.102.44, L3 100.105.64.115, L4 100.82.187.65, L5 100.101.8.76
(all `lineNpvs`, cred files `C:\Users\Lourdes Gunadasan\lineN.cred`). PVS serves `:5199`. See [[pvs-line2-deploy]],
[[pvs-line5-deploy]], [[pvs-manual-feeder-load]].
