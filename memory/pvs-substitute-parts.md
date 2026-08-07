---
name: pvs-substitute-parts
description: A part number with a slash in ProductBOM/feeder list = a substitute; PVS part verification must treat slash-separated as acceptable alternatives.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-07T01:50:59.601Z
---

**Domain rule (from Danial, 2026-08-07):** in **ProductBOM / the feeder list**, a part number that contains a **slash (`/`)** denotes a **substitute part** — the slash separates the primary part from its acceptable substitute(s). A reel labeled with any of the slash-separated values is a valid match for that feeder position.

**Why it matters — PVS gap:** the verification part-match is a plain exact string equality:
`PartsMatch(a,b) => string.Equals(a.Trim(), b.Trim(), OrdinalIgnoreCase)` — duplicated in `FullScanSession.cs`, `ModelChangeSession.cs`, `PartsChangeSession.cs` (all in src/Pvs.Core/Verification). So a slash-substitute expected part (e.g. `PARTA/PARTB`) does NOT match a reel scanned as just `PARTA` or `PARTB` → PVS **false-rejects a valid substitute** and demands supervisor release. Part verification is the wrong-build interlock, so this is a real correctness gap.

**Fix (pending — get the exact format first):** centralize the three `PartsMatch` copies into one Core helper that treats a slash-containing part on EITHER side as a set of alternatives — split on `/`, trim segments, match if ANY segment of the scanned part equals ANY segment of the expected part (case-insensitive). Add xUnit tests. MUST confirm the exact delimiter format with a real example before implementing (spaces? leading slash? multiple substitutes like `A/B/C`?) — over-loose matching would weaken the interlock (accept a wrong part); too strict re-introduces the false reject. See [[sony-smt-parts-verification]].
