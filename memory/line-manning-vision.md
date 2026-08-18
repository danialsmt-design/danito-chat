---
name: line-manning-vision
description: "Planned: per-line camera presence + badge identity, fused in the MCS brain → is the line manned / running unattended / by whom."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T07:54:31.132Z
---

**Planned build (2026-08-18, Danial's idea):** a camera per line to know **is the line manned, by whom, and is it ever running unattended.** Spec: `Documents/Dantec/MCS/line-manning-spec.md`.

**Design (the settled shape):** **camera = anonymous PRESENCE** (person-detection in the operator-station ROI, on an edge box per line, 1–2 fps, debounced ≥15s, NO frame/face storage); **badge = IDENTITY** (PVS already logs operator UID scan-in + L2 supervisor actions — reuse it, do NOT face-recognize); **MCS brain = FUSION** (presence + badge + production/bph → manning state). Key output: **"line producing but UNMANNED N min" → WhatsApp the supervisor.** No face database — privacy + labor-relations matter.

**Fits the existing events→reason pattern:** edge box → `POST /api/manning` (new PVS endpoint) or a Manning table → MCS brain fuses with the badge log + production, exactly like it reasons over serial + stock today. Not built yet — build in the MCS Ai session ([[mcs-ai-agent-spec]]) after the parts/stock work settles. Rollout: 1 line pilot → fuse+alarm → all lines + shift coverage report.
