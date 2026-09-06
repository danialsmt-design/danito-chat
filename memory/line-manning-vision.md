---
name: line-manning-vision
description: "Per-line camera presence + badge identity fused in the MCS brain; face-ID prototype (InsightFace, no HALCON) built 2026-09-06 at MCS/vision/faceid — a deviation from the spec's no-face-recognition rule."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-18T07:54:31.132Z
---

**Planned build (2026-08-18, Danial's idea):** a camera per line to know **is the line manned, by whom, and is it ever running unattended.** Spec: `Documents/Dantec/MCS/line-manning-spec.md`.

**Design (the settled shape):** **camera = anonymous PRESENCE** (person-detection in the operator-station ROI, on an edge box per line, 1–2 fps, debounced ≥15s, NO frame/face storage); **badge = IDENTITY** (PVS already logs operator UID scan-in + L2 supervisor actions — reuse it, do NOT face-recognize); **MCS brain = FUSION** (presence + badge + production/bph → manning state). Key output: **"line producing but UNMANNED N min" → WhatsApp the supervisor.** No face database — privacy + labor-relations matter.

**Fits the existing events→reason pattern:** edge box → `POST /api/manning` (new PVS endpoint) or a Manning table → MCS brain fuses with the badge log + production, exactly like it reasons over serial + stock today. Not built yet — build in the MCS Ai session ([[mcs-ai-agent-spec]]) after the parts/stock work settles. Rollout: 1 line pilot → fuse+alarm → all lines + shift coverage report.

**Face-ID prototype (2026-09-06):** Danial asked for facial recognition (first HALCON/MVTec — not installed, no licence, no native face op — then open-source). Built at `Documents/Dantec/MCS/vision/faceid/` (`faceid.py` engine, `enrol.py`, `identify.py`): InsightFace buffalo_l (SCRFD + ArcFace) on ONNX Runtime CPU, Python 3.14; models in `~/.insightface/models/buffalo_l`. Stores only 512-d embeddings per badge UID in SQLite, never images. Emits the spec's `/api/manning` event plus an `identity` field. Verified on the bundled group photo: enrolled face 0.99, all others < 0.07 at threshold 0.45. **This contradicts the spec's "no face recognition" principle** — if it goes to the floor it needs PDPA written consent per operator and a face-height camera (hairnets/masks will defeat it). Laptop webcam: black = lens shutter closed; torn frames = DirectShow format glitch when the Windows Camera app shared the device → capture now opens Media Foundation first with dead-frame reopen.

**Status 2026-09-06 (parked, "apply later to my project"):** live run verified on the laptop webcam; 4 people enrolled in `faces.db` (Danial = uid `danial`; Danish/Sufi/Mammy under temporary ids `tmp-<name>` because no badges were at hand — delete + re-enrol under the real badge UID later). Pairwise similarity between the four ≤ 0.16 vs threshold 0.45. `requirements.txt` pins the deps (Python 3.14). `faces.db` holds biometric templates of real people — never copy it into a git repo or share it. To apply: run `identify.py --post http://<line>/api/manning` on the line's edge box, add the `/api/manning` endpoint to PVS, fuse in the MCS brain per the spec. Enrol batch: `python enrol.py --names "A,B,C"` (Space to capture each).

**Depth / dot-projector idea (shelved 2026-09-06):** Danial asked about an iPhone-style dot projector for liveness/depth. Verified Malaysian prices: RealSense D435 RM 2,490 (infiniteit), D435i RM 2,799 (Cytron) — my earlier RM 1,200 guess was wrong. DIY route = IR dot projector (~RM 100-300) + no-IR-filter camera(s) (OV9281 ~RM 100-150 each): option 1 = one camera, dot-regularity liveness check (~1 day); option 2 = two cameras active stereo via OpenCV (several sessions). Not bought, not started — Danial said "shelf it first".
