---
name: pvs-board-flow
description: "The two-side board flow: store→bare-board pack→first-side line→magazine (slip PO+MAG#)→second-side line→FCT. Which side is first is production-decided & sticky."
metadata:
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T08:47:11.802Z
---

**A Canon board is double-sided → it runs the SMT line TWICE, one pass per side.** (Taught 2026-08-18.)

**Which side is FIRST is decided by PRODUCTION per model — NOT by the A/B label — and normally does NOT change once set (sticky).** A/B is just a name; do not assume A=first. PVS should hold "first side" as a set-once, cached per-model fact (a one-time supervisor/production decision), not derive it.

**FIRST side = fed by the STORE.** The store issues bare boards to that line as **PACKS**. The pack barcode is **exactly the reel/parts barcode format = part# + UID** (UID ends `%`); the pack's **board qty comes from StockOuts by UID** (`FindStockOutQtyAsync`). So a bare-board pack is handled IDENTICALLY to a reel: scan part# then UID, dedupe the UID, lot from the StockOuts row, reconcile scanned packs to the lot's board target. This is why no sample label was needed — PVS already parses this. See [[pvs-bare-board-pack-capture]].

**SECOND side = fed by the FIRST-side line, NOT the store.** Completed first-side boards load into **MAGAZINES**; each magazine travels with a **Production Identification Slip** (printed by the "PO Monitor" DB app today — **MCS will take over printing/ownership**). Slip QR = **PO number + magazine no.** (e.g. `HC20796226000-MAG01`); the slip also carries model, PO qty, production date+qty, the **ROUTE**, and AOI/QC/REMARKS trace fields. The second-side line **SCANS the magazine slip QR at input** (one scan per magazine); the **board count = the slip QUANTITY**. So second-side supply is tracked by magazine slips, never StockOuts.

**The slip ROUTE encodes the line pairing.** e.g. `LINE (5-A) - (1-B) - FCT` = **Line 5 A-side FIRST → Line 1 B-side SECOND → FCT** (function test). Confirmed for **L307: Line 5 (A/first) → Line 1 (B/second) → FCT**.

**BOARD-INPUT COUNT = the FIRST machine, NOT M4.** For reconciling boards CONSUMED from the input (a magazine on the second side, or a bare-board pack on the first side), count at the **first machine** — that's where each board enters the line from the magazine/loader. This is separate from the LOT/output count, which stays M4 (last machine). So: input/magazine consumption = first machine; production output/lot completion = M4. **Line 3 EXCEPTION: use MACHINE 2 for the count** — L3's M1 is the JUKI (non-serial, gives no board count), so M2 is the first countable machine. (Serial: all lines M1–M4 except L3 M1=JUKI; see [[pvs-machine-skip-rule]].)

**Line sides known:** Line 5 = A (first; L307/L313), Line 1 = B (second; L307/L261/L313), Line 2 = B (L309/L311). **Line 3 & Line 4 sides not yet locked.** Layout also flags a line RENUMBER (today's Line 5 = previous Line 1) — confirm LINE1PVS/LINE5PVS hostnames still map to the physical lines before trusting pairings. Pairing is sticky per model.

**CROSS-LINE BUFFER = live WIP between a paired first/second line** = magazines the first-side line produced − magazines the second-side line consumed. A second-side line with no magazine available is **"waiting boards from Line X"** — a distinct downtime cause, separate from waiting for a reel or for store bare-board stock. Measurable/predictable from the magazine counts, not just operator-tapped.

Connects: [[pvs-bare-board-pack-capture]], [[pvs-system-brain]], [[pvs-mcs-coordination]], [[pvs-lot-supervisor-only]], [[pvs-model-naming]], [[mcs-material-control-system]].
