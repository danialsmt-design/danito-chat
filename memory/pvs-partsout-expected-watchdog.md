---
name: pvs-partsout-expected-watchdog
description: "Diagnostic rule — a running line MUST produce parts-outs; prolonged silence = serial comm/parse break, not calm."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-17T15:29:58.763Z
---

**Danial's rule (2026-08-17):** On a **running** line, parts-outs are **guaranteed** — every reel that empties fires one. So parts-out silence on a producing machine is NOT "nothing happened," it's an **alarm: a break in the serial parts-out path** (signal not arriving, or not being parsed — e.g. an `R2E03` format the regex misses).

**How to estimate the expected cadence (so silence can be judged):**
- `parts consumed = per-board usage (MountedPerBoard) × boards run`.
- each reel's capacity = its **UID's StockOut quantity** (what it held when loaded).
- `boards-to-next-parts-out for a feeder = remaining ÷ per-board`; with bph → expected time.
- If a machine keeps completing boards **well past** the point its soonest reel should have run dry, and **no parts-out was received**, the serial parts-out is broken.

**Apply as a standing watchdog:** per machine, if it's producing (bph>0, boards advancing) but has logged **0 parts-outs for longer than the predicted exhaust window**, raise a serial-parts-out-comm-break alarm — don't wait to be asked. This is what would have caught **L4 M1** (running, consuming, but 0 parts-out events = the `R2E03` parse/format break) automatically.

Related: parts-out message format + parser is `R2E03S###N####Z###` (see PartsOutDetector / SonyMessage.cs `R2` regex); the retire that consumes the reel is [[pvs-reel-retire-rank-rule]]; the stock model is [[pvs-spare-vs-loaded]].
