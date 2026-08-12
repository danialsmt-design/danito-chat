---
name: pvs-machine-skip-rule
description: "PVS rule — skip is supervisor-only; never auto-skip a machine even if its serial isn't answering."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-12T02:53:52.748Z
---

**Every line is 4 machines. All machines are serial (Sony) EXCEPT the JUKI** (Line 3 machine 1 = JUKI, manual
trigger, no serial port). Lines 1/2/4/5 are 4 Sony-serial machines.

**Rule (Danial, 2026-08-12):** the SUPERVISOR decides which machine to skip. PVS must **never auto-skip / auto-drop
a machine** — even if that machine's serial isn't answering. A non-responding Sony stays IN the parts check (shows
offline) until a supervisor explicitly skips it via the ⃠ Skip button.

**Why:** auto-excluding a silent machine would silently stop verifying its parts — the opposite of what a parts
interlock is for. A stopped/parked Sony still answers polls (A4E00/A4E02), and even a truly-dead one must stay in
the check. Skip is a deliberate supervisor decision, per-lot, and audited.

**How to apply:** `_skippedMachines` is written ONLY by `SetMachineSkippedAsync` (supervisor, L2+) + load/restore.
Nothing keys skip/exclusion off `online`/serial status. The ⚙ manual-feeder panel lists the line's REAL machines
(config), so the supervisor sees all 4 and chooses. See [[pvs-manual-feeder-load]], [[pvs-feeder-source-productbom]].

**Line 3 ports (discovered 2026-08-12):** M1 JUKI (manual, no port), M2=COM3, M3=COM6 (cell 3, reports
`..._Cell3.PW4`), M4=COM4 (physically last → the count source). COM5 is empty. M3 had been dropped from the config
during the L264 "cell 3 unused" work; added back so it's a full 4-machine line and the supervisor skips per lot.
