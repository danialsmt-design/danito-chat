---
name: pvs-count-two-counters-and-cap
description: "Sony mounters have TWO counters (never-reset serial report vs operator-reset panel); trusting the wrong one inflated a lot, now capped."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-06T00:46:21.977Z
---

**The gotcha:** a Sony mounter exposes TWO different production counters, and PVS was trusting the wrong one.

- **Production *report* counter** — the C1M `PC` ("completed PWBs") field PVS reads over serial. Operators do **NOT** reset it per lot; it counts cumulatively (lifetime / across lots). See [[pvs-c1m-machine-counter]].
- **Main-panel production *count*** — the operator-facing HMI counter that operators DO reset at each lot change. This is the trusted per-lot number, and PVS **cannot read it over serial**.

**The incident (2026-08-05, Line 1, L307-B, lot HC20787036000):** "trust machine count" adopted the un-reset report counter (825 panels) as the lot count against a 300-panel / 1200-board target → lot card showed 3300 boards and a phantom **+2100** row was written to DailyProductionCount. Real production was ~640 boards (160 panels). Fixed by re-anchoring to 160 and deleting the phantom DB row (row ID 2262) via `dbsvc` (db_datawriter) — see [[parts-control-db-write-access]].

**The fix (Tier 1, deployed to all 3 lines 2026-08-06, commit 28a22e9):** `AdoptLotCount` now **refuses** to adopt a machine count beyond the lot target (+10% slack), refuses when the target is unknown (DB-down window) instead of falling open, and audits every adopt/reject/force-end/seed as VerificationRecords. `/api/lot/adopt` is now supervisor-badge-gated. `LotProgress` exposes `overTarget`.

**Operator agreement:** Line 1 operators will reset the report counter at each lot change going forward. **Why it matters:** even with that habit, the cap is the backstop for a missed/late reset.

**Next (not yet built):** Tier 2 = delta-based adoption — baseline the report counter at lot start (persist `pcBaseline` in lot-progress.json), trust only its *increment*. That removes the false "operators reset the report" assumption entirely. Part of the robustness roadmap (integrity → accuracy → traceability); see also the inventory audit finding that C1Z per-feeder pickup counts are the unused ground-truth for consumption.
