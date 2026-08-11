---
name: pvs-bare-board-pack-capture
description: "PVS planned feature — scan bare-board PACK labels (lot#+unique ID) at input, reconcile to lot size."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-11T15:32:46.227Z
---

Planned PVS feature (design agreed, NOT built — waiting on a sample label from Danial, promised 2026-08-12).

**What it is:** capture bare-board input by scanning **pack labels**, not individual boards. Each pack is a fixed
qty of boards (example: 50). The label carries **lot number + unique ID**. So a 300-board lot in 50-packs = the
operator scans **6 unique IDs** (6 × 50 = 300).

**Behaviour agreed so far** (per-scan):
1. **Lot match** — label's lot number must equal the running lot; reject a pack from the wrong lot (this is the
   main point: stop wrong bare boards going in). Running lot is supervisor-set — see [[pvs-lot-supervisor-only]].
2. **Duplicate guard** — a unique ID can't be scanned twice for the lot (double-feed / re-scan blocked).
3. **Reconcile to lot size** — accumulate scanned packs toward the lot target; tell the operator when fully covered.
4. **Trace** — log every unique ID against the lot (foundation for board→lot→reel genealogy; reel UIDs already
   captured per feeder).

**Still open (resolve with the sample):**
- Pack qty ("50") is NOT on the label — decide source: fixed/config standard, per-model, or work in pack COUNT =
  lot size ÷ standard pack size. Plus remainder packs (lot 280 → 5×50 + 1×30): are short packs normal?
- Exact barcode layout of lot# vs unique ID (need a real scanned string to parse them apart).
- On-scan action: hard-block the line until the full board count is staged, or just warn/reconcile.

Ties into "first process" (A-side = bare boards in from supplier packs); B-side handling deferred.
