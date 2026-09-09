---
name: mcs-line-schedule
description: MCS line-schedule page — derived from open POs (no typed schedule); 2-day post-SMT buffer; fixed-line rule; PO line = lot.
metadata: 
  node_type: memory
  type: project
  originSessionId: 23949a3c-1b8f-4a24-a154-b03a7886ae86
  modified: 2026-09-08T06:39:28.798Z
---

New MCS page (agreed 2026-09-08): a **derived** line schedule. There is no
planning schedule anywhere; the only plan input is the Canon PO with its
delivery date (DeliveryDocuments, shown on po.html / /api/po/monitor).

Rules Danial fixed:
- **Deadline** = PO due date minus **2 days** (FCT/assembly/QC/pack after SMT).
- **Fixed line as much as possible**: a model stays on the line it last ran
  on (home line from production history); move only when that line cannot
  meet the date, and show the move as a warning, never silently.
- **One PO line = one production lot** (no splitting).
- Two passes per board (A side then B side); B cannot start before A ends.
- Schedule is advisory: PVS lot change stays supervisor-badge only
  ([[pvs-lot-supervisor-only]]); MCS plans, PVS executes.

**Why:** Raja Rao has no tool that says what runs where and when; overdue POs
are discovered late. Deriving from POs avoids a second data-entry burden.

**How to apply:** build it as a back-scheduler over open POs (outstanding
qty, rate per model/line from DPC history, current PVS lot per line), lanes
= Line 1..5, colour bars by parts completability. Related:
[[mcs-material-control-system]], [[pvs-board-flow]], [[pvs-model-naming]].
