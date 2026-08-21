---
name: pvs-unload-and-rule-trees
description: "PVS \"Unload all feeders\" feature + the rule-tree/guardian workflow now used for PVS functions."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-21T02:29:39.568Z
---

Two things built 2026-08-21, both deployed to all 5 lines.

**Unload all feeders** — for month-end (all parts return to store to count StockIn) or a completely new model.
Takes EVERY reel off the feeders. **"Emptied" = removed from the machine, remainder in the UID maintained** —
so it ONLY clears the feeder→reel mapping + stops tracking; it **never writes StockOut**, never touches the lot
or board count (a half-done lot resumes from its produced qty when new reels are loaded), never writes attrition
(a return is NOT a consume). Reversible. Endpoints: `POST /api/unload`, `POST /api/unload/restore` (both L2+
badge-gated). Code: `FeederReelStore.ClearAll()/RestoreLastUnload()` (writes `inventory.json.unload-backup.json`),
`SessionCoordinator.UnloadAll()/RestoreLastUnloadAsync()`, both endpoints in Program.cs. UI: 🗑 **Unload line**
button on verify.html → badge-gated, two-tap confirm modal → return manifest (UID·part·qty) + CSV + restore.
Guardian-audited: no INVARIANT/MUST-NOT violated (sync-write path is starved because `MachineInventory.Clear()`
leaves zero tracked feeders). Distinct from the parts-out **retire** path (that one DOES consume + write attrition).

**Rule-tree workflow** — `docs/pvs-rule-trees.md`: one decision tree per PVS function
(Trigger→Gate→Confirm→Action→INVARIANTS→MUST-NOT→Output→Reverse). Write the tree BEFORE the code. Two watchers:
(1) a **hook** — `PVS/.claude/hooks/rule-tree-guard.cjs`, registered PostToolUse Write|Edit in the schedulle
`.claude/settings.json` — reminds on every PVS-source edit and flags a StockOut/attrition write inside Unload code;
(2) a **guardian agent** that audits code vs the tree. Only Unload's tree is written so far; the rest are stubs.
See [[pvs-system-brain]] · [[pvs-rules-bible]] · [[pvs-spare-vs-loaded]].

**verify.html declutter** (same day): Board-input + Standby cards collapsed behind one 🛠 toggle button (live
counts kept on the button, state persists in localStorage). Deployed to all 5.
