---
name: pvs-feeder-source-productbom
description: "PVS rule — feeders come from ONE source (DB ProductBOM), read fresh not cached; total feeder parts must match the Canon BOM regardless of how many machines/cells run."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T09:53:01.308Z
---

Danial's firm rule (2026-08-11): **all feeder data in PVS comes from ONE source — the DB `ProductBOM`** — read **fresh from the DB**, never from a local cache that can drift. This applies to the feeder **list** (shift-scan checklist / `FeederList`/`_expected`), the machine **inventory**, **parts-change**, and **lot-change** — they must all agree because they share the one source.

**And:** the **total number of feeder parts on a line must match the Canon BOM** — *irrespective of how many machines/cells the line is running*. If a line runs fewer machines (e.g. Line 3 with Cell 3 dropped), every Canon-BOM part must still be accounted for across the used machines, or PVS flags it (final confirmation that no part is missed).

**Why (the Line 2 incident, 2026-08-11):** the feeder list showed parts at **F109-118** while `/api/inventory` correctly showed **F125-134** (same parts, +16). Root cause: `ProductBOM.SupplyPosition` had been **updated** (positions moved to F125), and `/api/inventory` reads it fresh (`GetFeederMapAsync`, `Position.Number` = correct). But `_expected` (the feeder list) was served from a **stale local cache** — `model-cache.json` / `feedermap-cache.json` — saved before the update, and the pinned/loaded path never re-read the DB. The DB read is FAST (~0.3s), so there is no reason to trust the cache over it. Fixed immediately by deleting those two cache files → PVS reloaded `_expected` fresh from ProductBOM (F125, matched inventory).

**Code fix still to build:** always (re)load `_expected` from the DB ProductBOM on model-load and **on shift-scan** (the cache is a last-resort only if the DB is truly down; it must not override a reachable DB); and add the **Canon-BOM total-part reconciliation** as a final confirmation at lot/model select (warn/override/block behavior TBD — see [[canon-bom-source-of-truth]], [[pvs-lot-supervisor-only]]). Note `f.Position.Number` (SupplyPosition) is the correct feeder identity, NOT the cache's `f.Feeder`.
