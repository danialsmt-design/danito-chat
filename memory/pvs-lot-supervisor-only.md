---
name: pvs-lot-supervisor-only
description: "PVS must be passive on the lot lifecycle — lot change, lot end, and set-count are supervisor-badge ONLY; PVS never auto-follows/auto-ends/auto-writes a lot."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-12T03:39:19.994Z
---

Danial's firm policy (2026-08-11): **the lot lifecycle is supervisor-controlled only.** PVS must NOT:
- **auto-follow** the DB's current lot (GetCurrentLotAsync) and bind/switch to it,
- **auto-end** a lot (not on a machine model change, not on reaching target),
- **auto-write** production-count rows under a lot it picked itself.

Only a **supervisor (L2+ badge)** may: **select/change the lot**, **end the lot** (force-end), and **set/force the production count**. PVS's automatic jobs are: **count boards against the supervisor's lot** (holding that count through a machine reset — see [[pvs-count-two-counters-and-cap]]) and **manage stock-out quantities** (`syncStockOuts` — reel consumption; this stays ON, it's PVS's job).

**Config levers (immediate mitigation 2026-08-11):** `writeProductionCount` turned OFF on Line 1 to stop the phantom "PVS (auto)" DailyProductionCount writes; `syncStockOuts` left ON (stock-out qty is PVS-managed). The flag is all-or-nothing, so the proper code fix is to **re-enable count writing but gate it to a supervisor-set lot** (so a supervisor force-set still records, but PVS never auto-writes an auto-detected lot).

**Why:** the L307 incident (Line 1, 2026-08-11) — DailyProductionCount showed lot HC20790530000/L307 written every 5 min stamped `SenderIp='PVS'`, `OperatorName='PVS (auto)'`, while the supervisor was actually running L313/HC20789511000 (which had ZERO rows). PVS's auto-lot-follow + writeProductionCount=ON had bound and written a stale lot with no supervisor action. The running lot never got recorded.

**When the model changes but the supervisor hasn't ended/changed the lot:** PVS keeps the old lot and **flashes the program-mismatch banner** (built 2026-08-11: `/api/status.programWarn` when online machines disagree on model) to prompt them — it must NOT silently reset. A **skipped** machine is excluded from that alarm and from program-verify (2026-08-12) — see [[pvs-machine-skip-rule]].

**Sticky-lot fix (2026-08-12):** `SupervisorLotOnly` (LineConfig, default true) gates the DB-lot fallback in BOTH lot paths. The auto-detect path was guarded, but the **pinned-model path** (`VerifyAndTrackPinnedModelAsync`) was NOT — it fell back to the production system's `GetCurrentLotAsync` every cycle, overwriting a supervisor's fresh same-model lot pick and forcing a needless force-end (Line 2, L311). Both paths now honor the flag. **Normal same-model lot change = just select the new lot on ⚙ (resets count to 0); no force-end.** Force-end is only for closing a lot without starting the next.
