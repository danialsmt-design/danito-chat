---
name: smt-line-physical-layout
description: "Physical SMT line layout (13 stations, right-to-left flow) + Sony supply-station numbering rule (1xx=cassette, 5xx/6xx=tray) and what a CELL number actually is."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 0705d0b6-4b14-4d7a-b2fe-0b8c0a03a2bb
  modified: 2026-08-21T02:07:05.666Z
---

Source: Danial's "5 LINES SMT LAYOUT (RIGHT TO LEFT FLOW)" drawing + Sony manual `MAN00000743_SI-F_OPT.pdf` (SI-G/SI-F Communication System, 182pp, at `Downloads/comm sony/`), both supplied 2026-08-20.

## Line layout — 13 stations, flow RIGHT to LEFT

Board enters at station 1 (far right) and exits at 13 (far left). Numbering runs with the flow.

| # | Equipment | Width |
|---|---|---|
| 1 | Right-to-left **loader** | 1,100 |
| 2 | CONV 500mm | 500 |
| 3 | **PCB cleaner** | 1,250 |
| 4 | **3D SPI — Koh Young** | 1,350 |
| 5 | **Printer — Sony SIP750** | 1,700 |
| 6 | **MOUNTER Sony F130** | 1,650 |
| 7 | **MOUNTER Sony F130** | 1,650 |
| 8 | **MOUNTER Sony F209** | 2,100 |
| 9 | **MOUNTER Sony F209** | 2,100 |
| 10 | CONV 500mm | 500 |
| 11 | **Oven, 10-zone Eightech** | 4,200 |
| 12 | CONV 1000mm | 1,000 |
| 13 | **AOI — Jutze CON500mm** | 1,200 |

**So M1/M2 are F130s and M3/M4 are F209s** — the two machine types are not interchangeable positions. Per line ≈ **19.15 m**; five lines ≈ **95.75 m** excluding spacing. Clearances: operator (front) 1,000mm, maintenance (back) 1,000mm, ceiling 1,500mm. Line height incl. tower light ≈1,650mm. Max PCB width 500mm.

**PVS sees only stations 6–9.** Cleaner, SPI, printer, oven and AOI are invisible on serial — a stop caused by any of them shows up only as board rate → 0. See [[pvs-partsout-expected-watchdog]].

### ⚠ Two things in the drawing NOT yet confirmed with Danial (asked 2026-08-20)
1. **Line renumbering.** The drawing labels each line with a "(PREVIOUS LINE n)" — and it is a **reversal**: new 5←old 1, new 4←old 2, new 3←old 3, new 2←old 4, new 1←old 5 (**new = 6 − old**). Footnote: "Line order has been rearranged as requested." **UNKNOWN whether this is already done, or a planned/proposed layout.** If it has happened it is a serious data-integrity issue — every line-tagged row, COM-port map, PC hostname (LINE1PVS/LINE2PVS/LINE5PVS) and prior memory note may use the OLD numbering. **Do not assume either way; establish per source which scheme it uses.**
2. **Line 3 composition + SPI/printer order.** Line 3 shows **MOUNTER JUKI RS1 (1,890mm) at station 5**, where every other line has the printer — implying Line 3 has no printer and five mounters, which is implausible; its SPI width also reads 1,890 (others 1,350), looking like a copied cell. Separately, **SPI at 4 sits BEFORE printer at 5** on every line, which is backwards for solder-paste inspection. Both may be drawing errors. Line 3's JUKI being *first* in mounter order IS consistent with PVS calling it **M1** ([[pvs-machine-skip-rule]]).

## Sony supply-station numbering — AUTHORITATIVE (manual p.95)

> Supply station number: `1xx` = **cassette**; `5xx`, `6xx` = **tray**

**The older SI-E series spec differs** (`MAN00000188_SI-E2000_OPT.pdf` p.73): `1xx`, **`2xx`** = cassette; `5xx`, `6xx` = tray. So the E-series has a **2xx cassette range that the SI-F spec omits** — allow for it in any range-based rule rather than treating 2xx as unknown.

So the number itself encodes the feeder type, and the front/rear split follows it. This **confirms** the guess in `SupplyPosition.cs` that `[F]116 (F)` is a front tape cassette and `[F]501 (R)` is a rear tray — it is a **rule, not a coincidence**. Note **`6xx` is also tray** and PVS has likely never seen one.

**Actionable:** `SupplyPosition.Parse` derives side ONLY from the trailing `(F)`/`(R)` marker and returns `Unspecified` when absent (the `F12`/`Z12` machine-1 forms). It could fall back to the number rule. See [[pvs-feeder-source-productbom]].

Related manual facts:
- **Supply station offset (multi-origin) = 0–139**, and *"you cannot specify offset across the front and the rear part of the supply station"* → front and rear are **hard-partitioned**.
- **Tray unit is an option bit** (bit-5 of the head option), i.e. not every machine has one.
- Cassette I/F port number 1–8 per supply station.

## What a CELL number is (manual pp.83, 91, 95)

**Cell number = the machine's identity within the line** — 1 to 12 in the PWB data file, 1 to 4 in the supply-allocation file. Mount-step and supply-allocation records are keyed by cell#+head#.

Decisive line: *"It can be downloaded if it matches the logical number of each cell device."* → **a program only loads onto a machine whose configured cell number matches.** That is exactly why machine# must equal the reported CELL#, and why a cross-wired COM port shows up as a mismatch rather than as garbage — see [[pvs-port-cell-alignment]]. Also: heads are 1–4 per machine, each with a type (high-speed / odd-shape / multifunction) and option bits.
