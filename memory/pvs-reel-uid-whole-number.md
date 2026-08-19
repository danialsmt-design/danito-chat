---
name: pvs-reel-uid-whole-number
description: "Reel UID = the WHOLE number, never truncated to leading/partial digits. Truncated (0100-) vs full (880100-) split creates phantom-vs-real reel confusion."
metadata:
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-19T03:30:13.481Z
---

**Danial's rule (2026-08-19): "for the UID take the whole number, not the first digits only."** A reel UID must be captured/stored/matched as the COMPLETE number — never a truncated prefix/partial.

**Why:** part **VR8-5880-100** had reels under TWO UID formats — old **`0100-XXXX%`** (truncated) and new **`880100-XXXX%`** (whole). `0100-…` is `880100-…` with the leading `88` dropped. Because the same part appears under two encodings, paper stock and the loaded reel look like two different reels.

**Worked case:** StockOuts showed **`0100-E384%` = 5,509** on Line 1 (issued 18-08 18:28) as if it were stock, while the reel physically on Line 1 M2 F118 was **`880100-G429%`** (5,456). The operator said `0100-E384%` was "never loaded" — CORRECT: it's a **phantom row** (issued on paper, never on a machine). PVS did NOT write it — PVS tracks the real loaded UID (`880100-G429%`); the truncated `0100-E384%` row came from the issue/entry side. (I had first wrongly attributed the 18:28 write to a PVS sync — the loaded-UID check disproved that.)

**How to apply:**
- When reconciling/matching a reel, use the WHOLE UID; treat a `0100-…` as the truncation of `880100-…` for the SAME serial (but a different serial is a different reel — e.g. `0100-E384` ≠ `880100-G429`).
- Current PVS scan DOES capture the full UID (F118 held `880100-G429%` correctly) — the truncated `0100-` values are legacy/issue-side, not current PVS scans. Don't assume a PVS parse bug without checking the loaded UID first.
- The `0100-` truncated legacy rows are a data-integrity source of phantom stock → flag to the MCS/issue side (COORDINATION.md) to stop minting the short form, and zero confirmed phantoms (backup + Danial's go).

See [[pvs-spare-vs-loaded]], [[reading-true-stock]], [[pvs-mcs-coordination]], [[pvs-board-flow]] (board-pack UIDs use the same reel-UID format).
