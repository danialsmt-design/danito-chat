---
name: pvs-lot-supervisor-only
description: "PVS must be passive on the lot lifecycle — lot change, lot end, and set-count are supervisor-badge ONLY; PVS never auto-follows/auto-ends/auto-writes a lot."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T04:15:53.873Z
---

Danial's firm policy (2026-08-11): **the lot lifecycle is supervisor-controlled only.** PVS must NOT:
- **auto-follow** the DB's current lot (GetCurrentLotAsync) and bind/switch to it,
- **auto-end** a lot (not on a machine model change, not on reaching target),
- **auto-write** production-count rows under a lot it picked itself.

Only a **supervisor (L2+ badge)** may: **select/change the lot**, **end the lot** (force-end), and **set/force the production count**. PVS's only automatic job is to **count boards against the supervisor's lot** (and hold that count through a machine reset — see [[pvs-count-two-counters-and-cap]]).

**Why:** the L307 incident (Line 1, 2026-08-11) — DailyProductionCount showed lot HC20790530000/L307 written every 5 min stamped `SenderIp='PVS'`, `OperatorName='PVS (auto)'`, while the supervisor was actually running L313/HC20789511000 (which had ZERO rows). PVS's auto-lot-follow + writeProductionCount=ON had bound and written a stale lot with no supervisor action. The running lot never got recorded.

**When the model changes but the supervisor hasn't ended/changed the lot:** PVS keeps the old lot and **flashes the program-mismatch banner** (built 2026-08-11: `/api/status.programWarn` when online machines disagree on model) to prompt them — it must NOT silently reset. Implementation of the auto-follow/auto-end removal is queued with the [[pvs-connection-guardian]]-style consolidated rollout.
