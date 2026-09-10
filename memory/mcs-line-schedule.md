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
- **Which line runs which side is the PLANNER's call, not a hard rule** (Danial 2026-09-09, when I
  tried to hard-code "Line 5 = A side"): default = history (home line per model-side; when moving,
  prefer lines that have run that side at all). A per-line side restriction exists in settings
  (`LineSides`) but is EMPTY by default — only the planner sets it.
- **Model·side → line table** on the page (Danial 2026-09-09): one row per model-side, planner picks a
  line or "history"; stored as `HomeLines` {"L307|B":"1"} in settings (`POST /api/schedule/home`); a
  planner line beats the history home line (PlanItem.HomeSource = planner|history|none).
- **Non-Canon jobs** (Danial 2026-09-09): planner types lot ref + model + side (A/B/A then B/B then A/
  single) + qty + "SMT done by" date (+ optional line/note) on the page; stored under SchedulePlan key
  "manual" (`POST /api/schedule/manual`, `/manual/delete`); planned like a PO with buffer 0 (the date IS
  the SMT finish day); progress = DPC counts under that LotNo; rate = model history else default.
- **L347 is a PRE-PRODUCTION lot/model** (Danial 2026-09-09): setting `PreproductionModels` (default
  ["L347"]) keeps it out of the per-line utilisation measurement (it dragged Line 4 to 0.54) and out of
  the "lines that run this side" history; its own lots still plan at their own measured pace, tagged
  "pre-prod".
- Drag in TIME as well as line: dropping a bar earlier on a line = priority 1 on that line (run it
  first), later past everything = back to due-date order; another line = pin. Priority 0 clears.
- Advisory only: PVS lot change stays supervisor-badge only ([[pvs-lot-supervisor-only]]).
- **Operator working time (Danial 2026-09-09):** Mon–Fri 07:30–19:30 and the lines STOP at shift end.
  19:30–21:00 is overtime; **no plan on Saturday**. OT and Saturday exist only per DAY and can be
  per LINE (Danial 2026-09-09): click the day in the timeline header → day panel with an "All lines"
  row + one row per line → `POST /api/schedule/day {date, line?, ot|work|off|clear}`. Stored in
  SchedulePlan as "yyyy-MM-dd" (all lines) or "yyyy-MM-dd@N" (line N) in OvertimeDays / ExtraWorkDays /
  OffDays; `WorkCalendar(s, line)` resolves them. On an OT day that line's deadline moves to 21:00.
  Suggestions for late lots target the lot's own line ("Apply on Line N").
  Fixed breaks (lines stop): lunch 12:00–12:45, tea 15:30–15:45 (setting `Breaks`, cut out of every
  working window, so a shift = 11 h of work); bars show the gaps.
- **Manpower rule (Danial 2026-09-09):** 2 operators per running line; 1 line leader for every 3 running
  LINES (5 lines = 10 ops + 2 leaders). Bars on the Gantt are drawn per working stretch (PlanItem.Segments):
  stop at shift end / OT end, resume next working morning. Page lists it per day + per OT window; late lots get
  a suggestion (OT on these days / work this Saturday, with the manpower) the planner can Apply or ignore.
- Page also has a "Material shortage ahead" section (bottom) + a Parts-short tile: per scheduled
  model, the completability short parts with "needed from" = that model's first planned start;
  clicking a lot lists the short part numbers in its popup. Horizon can be 1–4 days (day stretches, hour marks).
- **Shortage rule (Danial 2026-09-09): ignore Canon invoices.** Short = needed − ACTUAL balance, where
  actual balance = store (StockIns.RemainingQty) + loaded feeders + reels at the line (= StockOuts
  latest-row-per-UID qty). InLineInv is a full subset of StockOuts (100% UID overlap) — never add it
  on top or it double counts. Parts Control checks ONE lot vs store, the page adds up every open lot of
  the model — that is why they can disagree (L264 / VC8-8380-106: 3,960 needed vs 3,632 balance).

Domain facts learned from the data:
- Home lines are per SIDE and the two sides run on different lines in a pipeline:
  L307 A→L5, B→L1; L309 A→L4, B→L2; L264 A+B→L3; L313 A→L5, B→L1; L311 A→L1/L2, B→L2/L3;
  L347→L4; L254 = single "Full" pass on L3. Second side starts hours after the first
  (magazines flow across) — modelled as a 2 h lag, not "after A completes".
- Side order per model comes from history (L311 AND recently L264 run B first).
- Running rate ≈ 150–280 boards/h while producing (5-min DPC buckets; bulk manual
  rows with Quantity>60 excluded); gross rate is far lower and erratic → plan uses
  running rate × utilisation. Utilisation is MEASURED PER LINE (Danial agreed 2026-09-09):
  within-run output buckets ÷ slots between first and last output of the day, break slots
  excluded, last 30 d, ≥5 runs (Sep-2026: L1 0.80, L2 0.70, L3 0.62, L4 0.54, L5 0.69, all 0.67);
  mode "fixed" or a line with no history uses the global factor (0.7). Breaks are NOT double
  counted: the running rate never contained them. Production runs 7 days incl. Sunday;
  night shift rare (setting, default off).
- **Monitor quirk:** `daiya.po` / `lotSize` / `totalOutput` are TODAY'S lots on the line ADDED TOGETHER
  ("HC…539000, HC…335000", 1200, 88) — not one lot. The current lot = LAST entry of `daiya.lots[]`
  ({lotNo, model, target, boards}); the parser uses that. A comma-joined lot string is still split and
  each open PO queued in order (safety net). Caught 2026-09-09 when L4 got a phantom 1,112-board lot
  and one PO was planned twice.
- "Running now" comes from the NAS line monitor `nas-daiya` :8899 `/api/all`
  (env `LINE_MONITOR_URL`); its line IP list was stale for L1/L3 on 2026-09-09, so the
  planner falls back to the DB's last production bucket (4 h window) for unreachable lines.
- Settings + overrides persist in `[MCS].dbo.SchedulePlan` (created via `sa`; the app
  login cannot CREATE TABLE in the MCS catalog).

**Why:** Raja Rao had no tool saying what runs where and when; overdue POs were found late.

**How to apply:** don't re-derive; extend `Scheduler.Plan` (pure, 14 tests). Tests run
in the `dotnet/sdk:8.0` container on the NAS (`/volume1/docker/mcs-test`) — no SDK on the
laptop. Related: [[mcs-material-control-system]], [[pvs-board-flow]], [[pvs-model-naming]].
