---
name: pvs-component-decrement-bible
description: The governing truth for PVS feeder/component consumption — per-lot total usage is the bible; board count drives the decrement; HMI sync upkeeps it; lot-end verifies; exhaust learning calibrates.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-09-01T08:42:26.415Z
---

**The per-lot total component usage is the BIBLE behind every component decrement** (Danial, 2026-09-01). The whole feeder-consumption system is built around this single governing number.

**The truth hierarchy (top governs):**
1. **Bible — per-lot total usage** = `MountedPerBoard × lot board count`, per feeder/component. This is the authority the running decrement must reconcile to. At lot completion, total consumed for a component MUST equal this. (`MountedPerBoard` = mount points per board × panel factor.)
2. **Board count drives the decrement** — each machine decrements ITS OWN feeders off ITS OWN board-out (R0) signal, never M4-for-all (L3 JUKI/M1 rides M2). `remaining = start − MountedPerBoard × boardsApplied`.
3. **HMI manual sync upkeeps the board count** — PVS CANNOT read the machine counter over serial while producing (Appendix F: C1M/C1Z answer A4E00 during AUTO production — see [[pvs-c1m-machine-counter]]). So the operator keys each machine's Sony HMI count (in **PANELS**) and PVS corrects that machine's feeders by the difference. Mid-line machines (M1–M3) need no badge; the PCB-out/last machine (M4) is supervisor-gated (its count is the lot count's basis). On the monitor page; M4 is corrected from the lot card on the verify page instead.
4. **Lot-completion verify** — at lot end (before the machine reset destroys the counts), PVS compares each machine's board tally to the lot count and flags any component whose usage deviates from the bible. `/api/lot/usagecheck` live; written to `lot-usage/<lot>.json`; audited as `LotUsageCheck`.
5. **Exhaust learning (shadow)** — the actual parts-out is ground truth: if a reel empties earlier/later than predicted, that drift is the tell-tale the per-board count is off. PVS learns a per-part EWMA correction (`ExhaustCalibration`, `/api/calibration`) — SHADOW only, never rewrites a reel count.

**Hard invariants:**
- **Never lose a reel's count.** All learning/verification is READ-ONLY; reel balances change only by the board-out decrement, a genuine parts-out, or an operator/HMI correction — all reversible/audited.
- Board tally is **seeded to the lot's board count on restart/re-baseline** (`SeedBoardsApplied`), else a post-restart HMI sync would double-decrement the restored feeders.
- HMI shows **panels (PWBs)**, not boards — sync sends `panels` to `/api/lot/adopt` and `/api/machine/synccount`.

Built 2026-09-01, deployed all 5 lines. Files: `MachineInventory` (BoardsApplied/SyncToBoardCount/SeedBoardsApplied/ReelUsage), `ExhaustCalibration`, `SessionCoordinator` (SyncMachineBoardCountAsync/LotUsageCheck/RecordExhaustCalibration). Related: [[pvs-feeder-consumption-c1z]], [[pvs-count-two-counters-and-cap]], [[pvs-c1m-machine-counter]].
