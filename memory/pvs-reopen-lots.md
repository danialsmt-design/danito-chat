---
name: pvs-reopen-lots
description: "Per-line \"reopenLots\" config forces old/incomplete carry-over lots into the manual lot dropdown past the 7-day window."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-25T01:45:45.912Z
---

Built 2026-08-24. The manual lot dropdown (`GetLotOptionsAsync`) normally only lists a model's DeliveryDocuments lots that are non-Delivered AND `DeliveryDate >= today-7`. So an old incomplete lot (e.g. a July lot whose B-side was never run) can't be selected — it's past the 7-day window.

**Fix:** a per-line config list **`reopenLots`** (PO numbers) that force-includes those exact POs regardless of date. Still gated to the selected model + non-Delivered status, so ONLY the listed POs surface, nothing else. Query adds `OR PONumber IN (@r0,@r1,…)` inside the date-OR group; `LineConfig.ReopenLots`; coordinator passes it to `GetLotOptionsAsync(model, reopenLots, ct)`.

**First use (Line 1 only):** `reopenLots: ["HC20784402000","HC20784403000"]` — two **L307** July lots, A-side already done (300 each on Line 5 in July), B-side never run. Operator selects model L307 on L1 → both appear at the top of the dropdown → run the B-side. To retire: remove from the list (or they drop off once marked Delivered).

**Fleet note:** code + config deployed to **Line 1 only** so far; the other 4 lines still run the prior build (harmless — no reopen field). Sync the code to them next idle window (empty `reopenLots` = no change) to keep one build version. Related: [[pvs-lot-supervisor-only]], [[pvs-model-naming]], [[pvs-db-failsafe]].
