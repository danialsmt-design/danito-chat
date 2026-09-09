---
name: mcs-line-schedule
description: MCS line-schedule page (LIVE 2026-09-09) — derived from open POs; 2-day post-SMT buffer; home line per model-SIDE; PO line = lot; advisory; how it computes and where its inputs come from.
metadata: 
  node_type: memory
  type: project
  originSessionId: 23949a3c-1b8f-4a24-a154-b03a7886ae86
  modified: 2026-09-09T03:02:55.075Z
---

**LIVE 2026-09-09** on the NAS: `gmssmt1:8090/schedule.html` + `GET /api/schedule`
(`app/Schedule/*` in `Documents/Dantec/MCS/app`). No typed schedule exists; the
only plan input is the Canon PO delivery date (DeliveryDocuments).

Rules Danial fixed (2026-09-08):
- **Deadline** = PO due date − **2 days** (FCT/assembly/QC/pack), end of the day shift.
- **Fixed line as much as possible**: a model-**side** stays on the line it last ran
  on ("home line", from DailyProductionCount, last 60 d); it moves only when the home
  line cannot make the date AND the move actually makes it (else stays home, flagged late).
- **One PO line = one production lot** (no splitting).
- Advisory only: PVS lot change stays supervisor-badge only ([[pvs-lot-supervisor-only]]).

Domain facts learned from the data:
- Home lines are per SIDE and the two sides run on different lines in a pipeline:
  L307 A→L5, B→L1; L309 A→L4, B→L2; L264 A+B→L3; L313 A→L5, B→L1; L311 A→L1/L2, B→L2/L3;
  L347→L4; L254 = single "Full" pass on L3. Second side starts hours after the first
  (magazines flow across) — modelled as a 2 h lag, not "after A completes".
- Side order per model comes from history (L311 AND recently L264 run B first).
- Running rate ≈ 150–280 boards/h while producing (5-min DPC buckets; bulk manual
  rows with Quantity>60 excluded); gross rate is far lower and erratic → plan uses
  running rate × utilisation (setting, default 0.7). Production runs 7 days incl. Sunday;
  night shift rare (setting, default off).
- "Running now" comes from the NAS line monitor `nas-daiya` :8899 `/api/all`
  (env `LINE_MONITOR_URL`); its line IP list was stale for L1/L3 on 2026-09-09, so the
  planner falls back to the DB's last production bucket (4 h window) for unreachable lines.
- Settings + overrides persist in `[MCS].dbo.SchedulePlan` (created via `sa`; the app
  login cannot CREATE TABLE in the MCS catalog).

**Why:** Raja Rao had no tool saying what runs where and when; overdue POs were found late.

**How to apply:** don't re-derive; extend `Scheduler.Plan` (pure, 14 tests). Tests run
in the `dotnet/sdk:8.0` container on the NAS (`/volume1/docker/mcs-test`) — no SDK on the
laptop. Related: [[mcs-material-control-system]], [[pvs-board-flow]], [[pvs-model-naming]].
