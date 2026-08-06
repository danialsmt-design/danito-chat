---
name: pvs-c1m-machine-counter
description: "Sony C1M production-report read is rejected (A4E00) by Line 1 M4 — machine-side config, not a code bug."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-07-31T15:14:40.149Z
---

**✅✅ SOLVED 2026-07-31 22:41 — C1M WORKS. M4 returned CompletedPwbs = 28,596.** It was **OUR BUG**,
not the machine: we were sending `C1M000P` (a `P` separator with an EMPTY data name). The correct
whole-machine form is the **BARE `C1M000`** — no `P` at all (SI-F 6.2.2: *"If the data name is not
added, the overall machine status is loaded"*). Fixed in `MachineChannel.RequestProductionCount(now,
pwbName = null)`; reproduced twice with moving `CompletedPwbsAt` timestamps, `error: null`.
- `C1M000` (bare) → **works**. `C1M000P<program name>` → **A4E01** (so don't use the P form; the name
  format isn't what we guessed). Old `C1M000P` (empty name) → **A4E00**.
- **The reply is parseable ASCII — no ZIP/binary decoding was needed** for the `PC` field. The
  manual's "compressed by the ZIP format" warning did NOT block the serial read.
- ⚠️ **The report takes ~30–35 s to stream at 9600 baud.** It currently only lands via the *timeout
  salvage* path (channel timeout is 8 s, API wait was 10 s) — fragile. **NEXT SMALL FIX: raise the
  channel report timeout to ~45 s** so it completes properly instead of being salvaged.
- API now `POST /api/machinecount?machine=N&pwb=<name|auto>&waitMs=N` and returns `sent` / `error`
  (the A4/A5 reject code) / `program` — `MachineChannel.LastReportCommand` + `LastReportError` were
  added so a refusal says WHY. 172 Core tests green. Deployed to `C:\PvsLineApp\` (Pvs.Core +
  Pvs.Data + Pvs.LineApp DLLs; backup `C:\PVS_Backups\dll_20260731-223731`).
- ⚠️ **M1/M2/M3 still return `A4E00` even to the bare `C1M000`**, and also fail to answer `C3P`
  (their `program` came back empty) — so they are in a different machine STATE, not a protocol
  problem. M4 is the healthy link. Check those three at the HMI (emergency stop / a setup dialog
  open / axis operation — see the Appendix G analysis below).
**🎯 PER-MODEL / PER-LOT COUNT ALSO WORKS (2026-07-31 22:53) — Danial spotted that `C1M000` is the
ALL-MODELS machine total, not per model.** Adding the PWB data name scopes it:
`C1M000P<pwb name>` → **M4 returned 155**, versus 28,596 for the bare entire-machine form.
- **✅ CROSS-VALIDATED: the machine's 155 EXACTLY equals PVS's own independently-counted
  `/api/lot` `panels: 155`** (620 boards ÷ 4-up). Two separate measurement paths — our R0 event
  counting vs the machine's internal counter — agree exactly. **PVS's board counting is sound.**
- **✅ UNIT ANSWERED: the counter is PANELS (machine cycles), NOT child boards.** 155 panels × 4 =
  620 boards. Applies to the 28,596 entire-machine figure too.
- **NAME RULES (learned by experiment):** must be **byte-exact and WITHOUT the extension**.
  `L307 - B SIDE _Cell4` ✅ · `L307 - B SIDE _Cell4.PW4` → **A3** (= file not found) ·
  `L307-B SIDE _Cell4` (dash spacing changed) → **A3**. Machine names are inconsistent across cells
  (`SIDE_Cell1`/`SIDE_Cell2` vs `SIDE _Cell3`/`SIDE _Cell4`) — **never hand-type; always derive from
  `C3P`'s ProgramName minus the extension**, which is what `?pwb=auto` does. (A one-off `A4E01` was
  seen on this same name earlier the same evening, then it worked — treat A4xx as retryable state.)
- ✅ **ANSWERED by Danial 2026-07-31: the OPERATOR RESETS the production count at the END OF EVERY
  LOT, on ALL FOUR machines, and it is in the WRITTEN SOP.** So the per-PWB counter IS a per-lot
  count (that is why it matched 155 exactly), but the reset is a **human action** — expect it
  mistimed, doubled, or missed.
  ⚠️ **This KILLS my earlier "use deltas" advice** — a delta spanning a reset comes out NEGATIVE, or
  worse, small-and-positive in a way that silently hides production.
  **⇒ VALIDATION DESIGN (agreed):** PVS keeps its live R0 count; periodically read the machine
  counter and compare per lot. Machine ≈ PVS → healthy, log it. **Machine LOWER → a reset happened**
  (re-anchor PVS's baseline, record a reset event; do NOT treat the drop as production). **Machine
  HIGHER → PVS was blind and missed boards** (adopt the machine's figure, record the gap size).
  **NEVER silently correct — log every reconciliation with its reason**; those discrepancies ARE the
  data for [[mcs-material-control-system]].
  **The two counters fail in OPPOSITE ways** — PVS misses boards when off/serial drops but never
  resets; the machine survives PVS downtime but resets on a human action. Neither alone is
  sufficient; that is the whole point of cross-checking.
  **All 4 machines reset ⇒ 4 independent measurements on a COMMON BASELINE:** at lot start all four
  should read ~0 (a non-zero one = reset missed on that machine, catch it before the lot's data is
  polluted); during the lot the four should track each other (boards run M1→M4 in series — one far
  behind = stopped/bypassed; M1 ≫ M4 = boards entering but not completing); at lot end all four ≈ lot
  qty. Because it is SOP, a deviation is **auditable**, not just "someone forgot".
  **🔴 THE RESET DESTROYS THE EVIDENCE** — read the counter only AFTER the operator resets at lot end
  and the lot's final count is gone permanently. This, not drift, is the real reason for a **15-min
  heartbeat read**: it bounds the worst-case loss to the last 15 min of a lot instead of the whole
  lot. Reading "at lot end" alone loses the race whenever a human presses reset first.
  **Cadence:** lot start · lot end · shift change · after any PVS restart or machine offline→online
  (exactly when PVS was blind) · plus the 15–30 min heartbeat. A read takes ~30 s so it cannot be
  continuous.
- ⇒ **`C1Z` (per-supply-location report) should now be reachable the same way** — that's the
  per-feeder pickup/loss data for [[mcs-material-control-system]].

**✅ RECONCILIATION BUILT + DEPLOYED + LIVE-VERIFIED (2026-07-31 23:14).**
- **`Pvs.Core.Runtime.CounterReconciler`** (pure, 11 tests) — `Reconcile(machine, machineCount,
  pvsCount, previousMachineCount, tolerance=3, error)` → `ReconcileResult` with status
  **Agree · ResetDetected · PvsBehind · PvsAhead · NoRead**. Rules: a counter going BACKWARDS vs our
  previous read is the ONLY hard evidence of an operator reset and is checked FIRST (after a reset the
  machine-vs-PVS gap is huge and would otherwise read as PVS double-counting); machine > PVS =
  PvsBehind (`ShouldAdoptMachineCount`, the machine is right); PVS > machine with no reset evidence =
  PvsAhead → `NeedsAttention`, **never auto-corrected**. Also `CrossMachineSpread()` — all 4 reset at
  lot start so they share a baseline; a machine far behind was stopped/bypassed/never reset.
- **`Pvs.LineApp.Runtime.CounterReconcilerService`** — heartbeat timer (`LineConfig.ReconcileMinutes`,
  default **20**; first pass 2 min after start), reads ALL machines in parallel (separate COM ports),
  45 s read window, PWB name auto-derived from `C3P` ProgramName minus extension (never hand-typed).
  Appends every run to `C:\PvsLineApp\reconcile\reconcile-YYYY-MM-DD.jsonl`; remembers each machine's
  last reading in `reconcile-previous.json` **so a reset during PVS downtime is still detectable as a
  backwards step**. Observe-and-record ONLY — it never rewrites PVS's counters.
- API: `GET /api/reconcile` (recent runs + interval) · `POST /api/reconcile/run?trigger=` (~45 s).
- Also added `MachineChannel.BoardsSeen` and `SessionCoordinator.CurrentLotPanels`.
- **LIVE RESULT:** M4 `Agree` (machine 155 = PVS 155, pwb `L307 - B SIDE _Cell4`); M1/M2/M3 `NoRead`
  with `A4E00` surfaced in plain language instead of a silent null. 183 Core tests green. Deployed
  `C:\PvsLineApp\` (backups `C:\PVS_Backups\dll_20260731-223731` and `dll_20260731-231223`).
- **NOT yet wired:** event-driven passes at lot start / lot end / shift change / machine-back-online
  (currently heartbeat + manual only) — and no UI page; the data is in the JSONL + API.
- ⚠️ The 8 s channel timeout is still in place, so a read lands via the **salvage** path (`PC` is early
  in the report). Proven repeatedly, but raising the timeout to ~45 s remains the tidy fix — with the
  caveat that D0 messages are treated as report data while collecting, so a longer window means a
  longer period where a stray `C3P` reply could be swallowed.

Attempt to read each Sony mounter's own completed-PWB counter over serial (Production Report,
command `C1M`) to make the PVS lot count drift-proof. ~~**Blocked at the machine, not in our code.**~~

Live-tested on Line 1 M4 (the one machine whose link answers C3P): both `C1M000` and `C1M000P`
return `A4E00` (command rejected).

**🔴 DIAGNOSIS CORRECTED 2026-07-31 from the manuals — my earlier "not enabled on the link / possibly
vendor-locked / blocked in AUTO" reading was WRONG. Nothing needs enabling.**
`sif.txt` **Appendix G "Error Exclusive Control Table (SI-F Series)"** (~line 9154) is a state ×
command matrix; `C1M` is the 3rd column. In the AUTO Production block C1M reads **`Possible`** for:
AUTO Production Operation in progress, Pass-through, Alarm dialog displayed, Alarm occurring, Parts
Change (**orange**) dialog, PWB Select dialog. It is only blocked as **A5E02** (Parts Change **blue**
dialog) / **A5E01** (PWB data downloading) = BUSY-retry, and **A4E02** off-line. **None of those is
A4E00, and C1M is explicitly allowed DURING automatic production.** (This is where I went wrong: I
assumed C1M shared C1P's restriction. `e2000.txt` line 1079 blocks **C1P** in automatic run; line
1216 blocks **C1M** only in **EMERGENCY STOP**. Different commands, different rules.)
⇒ **A4E00 on C1M = machine STATE**: emergency stop, or an HMI setup/maintenance dialog or axis
operation open at the panel (the Appendix G block above AUTO — M/C data setting, axis operation,
origin return, height teaching, password/language dialogs all give A4E00).
⇒ **ACTION: just retry `C1M000` while M4 runs normally with nobody at the panel.** No Sony call.

**🔑 BIGGER FIND — `C1Z` = Production Report SUMMARY BY SUPPLY LOCATION (per FEEDER).** Family:
`C1M` entire-machine/by-PWB-file · `C1N` by nozzle · **`C1Z` by supply location** · `C1U` nozzle
changer (SI-G only); plus C1MC/C1NC/C1ZC/C1UC variants. Supply Location = the same feeder id as the
parts-out `Z` field. Report fields (8 digits each) include **#5 Number of Successful Pickups**, #6
Missed Pickup Errors, #7 Abnormal Pickup Errors, #8 Recognition Errors, #13 Parts-Out Stops, #16
Pickup Error Stops, #18 Power ON Time, #19 Operation Time (sec). ⇒ **the machine MEASURES parts
actually consumed per feeder, plus the loss broken into named categories** — far better than
inferring `boards × ProductBOM.Quantity`. This should be the consumption + loss foundation for
[[mcs-material-control-system]] instead of arithmetic on a board count. C1Z's error conditions are
stated as identical to C1M's, so if C1M answers, C1Z answers.
**UNVERIFIED:** that C1Z breaks those pickup fields down per location (inferred from the
machine-status field list) — test that first. Also the manuals are the **[Network]** variant while
PVS is on **serial**; C-commands have matched so far (C3P, C5RO proven) but **binary D0 + unzip over
RS-232C is unproven**, and that transport is the real build work.

The reader IS built and correct (per SI-E2000 Fig.7-9): MachineChannel.RequestProductionCount sends
`C1M000P`, collects the D0 ASCII report, acks lines A0 + closes A2, parses the `PC` field
(SonyProductionReport.CompletedPwbs). Fully isolated + 8s timeout — proven safe: real-time / parts-out
untouched during both reads (M4 stayed online, frames advancing). It will work the moment C1M upload is
enabled machine-side. Committed at db81d24. `/api/machinecount?machine=N` triggers it on demand.

The real lot-count drift (292 vs machine 300, ~8 boards) came from MY ~dozen redeploys in one day
(each restart blinds PVS ~10s), not normal ops. Normal fix = the anchor counter (survives restarts) +
optional DB re-seed of the anchor from DailyProductionCount at lot start. See [[pvs-shift-and-daily-report]],
[[sony-smt-parts-verification]].
