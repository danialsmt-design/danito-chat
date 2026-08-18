---
name: pvs-port-cell-alignment
description: "PVS machine# must equal the program CELL# it reports; a mismatch = cross-wired COM ports. Wake-ups include this check."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T07:42:57.647Z
---

**Rule (Danial, 2026-08-18):** Each Sony machine's program name carries its physical **cell number** — `...Cell3.PW4`, or `...FUL(MC1)_Cell1.PW4` (MC1 = machine/cell 1). **PVS's machine number MUST equal the cell number that port reports.** If PVS M1 reports Cell3, the COM ports are **cross-wired** — the cable PVS calls machine 1 is physically on machine 3.

**A wake-up is now a wake + cell-alignment CHECK:** after sending C3P (`POST /api/programs/query`), read each machine's `program`, extract `Cell(\d)`/`MC(\d)`, and flag any machine where cell ≠ machine#. Also flag OFF / skip / parked(no-cell). Make this the normal wake routine, per-line.

**L4 was cross-wired (found + fixed 2026-08-18):** PVS M1=COM3=Cell3, M2=COM4=Cell4, M3=COM5=Cell2(parked), M4=COM6=Cell1. Fixed by remapping `C:\PvsLineApp\line.config.json` machines: **M1→COM6, M2→COM5, M3→COM3, M4→COM4** — the `model`/`hasTrayFeeder` travel WITH the port (physical machine): M1&2 = F209/tray (Cell1&2), M3&4 = F130 (Cell3&4). Backup `line.config.json.bak-portfix-20260818`. Verified M1→Cell1, M3→Cell3, M4→Cell4 after restart.

**This resolved the "L4 M1 parts-out not triggering" saga** ([[pvs-partsout-expected-watchdog]], [[nfc-hid-reader-protocol]] unrelated): PVS's "M1" was physically machine 3 the whole time, so its feeder/parts/parts-out mapping was shifted. Not a serial/parser bug — a cross-wired config. L2, L3, L5 checked and correctly aligned. Serial machine layout + ports: [[pvs-machine-skip-rule]].
