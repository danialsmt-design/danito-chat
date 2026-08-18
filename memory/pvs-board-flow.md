---
name: pvs-board-flow
description: "The two-side board flow: store→bare-board pack→first-side line→magazine (slip PO+MAG#)→second-side line→FCT. Which side is first is production-decided & sticky."
metadata:
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T15:24:42.192Z
---

**A Canon board is double-sided → it runs the SMT line TWICE, one pass per side.** (Taught 2026-08-18.)

**Which side is FIRST is decided by PRODUCTION per model — NOT by the A/B label — and normally does NOT change once set (sticky).** A/B is just a name; do not assume A=first. PVS should hold "first side" as a set-once, cached per-model fact (a one-time supervisor/production decision), not derive it.

**FIRST side = fed by the STORE.** The store issues bare boards to that line as **PACKS**. The pack barcode is **exactly the reel/parts barcode format = part# + UID** (UID ends `%`); the pack's **board qty comes from StockOuts by UID** (`FindStockOutQtyAsync`). So a bare-board pack is handled IDENTICALLY to a reel: scan part# then UID, dedupe the UID, lot from the StockOuts row, reconcile scanned packs to the lot's board target. This is why no sample label was needed — PVS already parses this. See [[pvs-bare-board-pack-capture]].

**SECOND side = fed by the FIRST-side line, NOT the store.** Completed first-side boards load into **MAGAZINES**; each magazine travels with a **Production Identification Slip**. The second-side line **SCANS the magazine slip QR at input** (one scan per magazine); board count comes off the slip, never StockOuts.

**MCS REDESIGNED the slip (seen 2026-08-18) — the current format:**
- Header: `GLOBAL MANUFACTURING SOLUTION · Production Identification Slip`; **MAG number boxed** top-right beside a large **QR**.
- Fields: **PO** (e.g. HC20782775000), **SIDE = A / B** (NEW — the side is now on the magazine itself), **MODEL** (e.g. L309), **PO QTY** (e.g. 300), **CUST** = `Canon Opto (M) Sdn Bhd · LINE n` (replaces the old `LINE (5-A)-(1-B)-FCT` route string), **DATE + "N PCS / mag"** (NEW explicit per-magazine board count, e.g. **30 PCS/mag**), + AOI VI / QC sign-off (REMARKS dropped).
- **Per-magazine qty is explicit on the slip** (e.g. 30/mag → PO 300 = 10 magazines) — so second-side reconciliation reads 30/mag off the slip, no StockOuts lookup for magazines.
- **QR content CONFIRMED (MCS deployed 2026-08-18):** `PO: <po> | QTY: <total> | MAG: <n>/<total-mags> | DATE: <dd/MM/yyyy>` — e.g. `PO: HC20792053000 | QTY: 1800 | MAG: 3/10 | DATE: 18/08/2026`. **MAG n/total is the unique per-magazine key** (dedupe on it; `total` = magazines expected for the lot). PO + QTY (total) kept in their original positions so an existing parser still reads them; DATE disambiguates reprints/days.
- **⚠ Per-magazine board count is NOT QTY/total.** It is a value the **supervisor or store PIC INPUTS per magazine** (magazines can hold different amounts — a partial/remainder magazine is normal). The printed "N PCS/mag" on the slip is that input. So PVS must use the ENTERED per-mag qty, not a computed split. Source of that number for PVS: TBD — MCS/DB lookup by PO+MAG#, or operator keys it at the second-side scan (confirm with Danial).
- **PVS second-side parse/reconcile:** scan QR → PO (match running lot) + MAG `n/total` (dedupe by n, expected count = total) + DATE (guard reprints); per-magazine boards = the supervisor/store-PIC input; accumulate scanned mags toward the lot. Count-only, no interlock (same as bare-board packs).
- Old (PO-Monitor) format was: CUSTOMER `CANON · LINE (5-A)-(1-B)-FCT`, PO QTY 1200, per-slip QUANTITY 400, QR not-unique-per-mag, REMARKS present.

**The slip ROUTE encodes the line pairing.** e.g. `LINE (5-A) - (1-B) - FCT` = **Line 5 A-side FIRST → Line 1 B-side SECOND → FCT** (function test). Confirmed for **L307: Line 5 (A/first) → Line 1 (B/second) → FCT**.

**BOARD-INPUT COUNT = the FIRST machine, NOT M4.** For reconciling boards CONSUMED from the input (a magazine on the second side, or a bare-board pack on the first side), count at the **first machine** — that's where each board enters the line from the magazine/loader. This is separate from the LOT/output count, which stays M4 (last machine). So: input/magazine consumption = first machine; production output/lot completion = M4. **Line 3 EXCEPTION: use MACHINE 2 for the count** — L3's M1 is the JUKI (non-serial, gives no board count), so M2 is the first countable machine. (Serial: all lines M1–M4 except L3 M1=JUKI; see [[pvs-machine-skip-rule]].)

**Line sides known:** Line 5 = A (first; L307/L313), Line 1 = B (second; L307/L261/L313), Line 2 = B (L309/L311). **Line 3 & Line 4 sides not yet locked.** Layout also flags a line RENUMBER (today's Line 5 = previous Line 1) — confirm LINE1PVS/LINE5PVS hostnames still map to the physical lines before trusting pairings. Pairing is sticky per model.

**CROSS-LINE BUFFER = live WIP between a paired first/second line** = magazines the first-side line produced − magazines the second-side line consumed. A second-side line with no magazine available is **"waiting boards from Line X"** — a distinct downtime cause, separate from waiting for a reel or for store bare-board stock. Measurable/predictable from the magazine counts, not just operator-tapped.

Connects: [[pvs-bare-board-pack-capture]], [[pvs-system-brain]], [[pvs-mcs-coordination]], [[pvs-lot-supervisor-only]], [[pvs-model-naming]], [[mcs-material-control-system]].
