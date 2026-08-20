---
name: floor-ai-manager-agent
description: "Floor Ai — the 24/7 shop-floor manager agent, spun out 2026-08-19 as a THIRD workstream/session; read-only by design; inherited the two guardian jobs from MCS Ai."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0705d0b6-4b14-4d7a-b2fe-0b8c0a03a2bb
  modified: 2026-08-19T15:20:53.777Z
---

**Floor Ai** = the always-on shop-floor **manager** agent. Created **2026-08-19** as a **third** workstream, split out of this (PVS/*schedulle*) session. Folder `Documents/Dantec/FloorAi` with its own `CLAUDE.md` + `BUILD-BRIEF.md`; runs in its **own Claude session** (which starts with NO memory — the brief is deliberately self-contained). **Status: specced, not built.**

**The three workstreams now:** PVS (`Dantec/PVS`, this *schedulle* session) = line runtime/edge sensing · MCS Ai (`Dantec/MCS`, NAS session) = DB/schema + entry pages, the front door that creates rows · **Floor Ai** = monitoring, judgement, reporting. See [[pvs-mcs-coordination]] — the ledger `MCS/COORDINATION.md` was updated to three workstreams and is still the only cross-session channel.

**Why a third session, not a widening of MCS Ai:** MCS Ai is mid-flight on the DB rehost + cutover, so the manager would get built in the gaps; and the manager's unusual discipline (read-only, event-driven, token-prudent) is easier to hold in a session that does nothing else. **One** manager brain, not two — two always-on agents watching the same floor would contradict each other, the exact problem the ledger exists to solve.

**Scope MOVED from MCS Ai → Floor Ai (2026-08-19, Danial's call):** (1) connection guardian, (2) stock-accuracy guardian. `MCS-Ai-BUILD-BRIEF.md` now carries a SUPERSEDED banner and hands both over; it is kept only because Floor Ai inherits its verified infra (NAS access, DB creds + date traps, WhatsApp bridge, line IPs/endpoints, guardrails). Supersedes the build instruction in [[mcs-ai-agent-spec]].

**THE governing constraint — read-only.** Floor Ai has **no row in the ownership map** by design, so it can never collide with PVS or MCS Ai. It may poll/read/compute/judge/report/escalate autonomously. It may **never** write to a DB, command a line, or broadcast to anyone but Danial. Anything more: Floor Ai *proposes* → Danial approves → **the owning workstream performs the write.** Floor Ai never becomes the hand that acts. Same principle as [[pvs-lot-supervisor-only]], held harder because it sees more.

**The layer model (the key reframe):** a downloadable/fine-tuned LLM is NOT the monitor — it has no eyes. Layer 1 sensing (serial, scanners, badges, cameras) + layer 2 state (DB) are ~90% of the work and contain zero AI; PVS already built most of it. The LLM only occupies layer 3 (judgement) and 4 (voice). **When Floor Ai is blind, the fix is a sensing gap, not a better prompt** — hence the Sensing Gap Register in its brief. Local small models are for narrow edge jobs only (downtime-reason classification, on-device person detection); the manager brain is an API model. See [[pvs-system-brain]].

**Phases** (each soaks and is signed off before the next; widening authority is a recorded Danial decision): **0** shift briefing at 07:35/19:35 to Danial via WhatsApp, zero authority, 2-week soak whose real purpose is finding sensing gaps → **1** connection guardian → **2** stock-accuracy guardian → **3** production management (output vs target, downtime analysis, WIP/board flow, over-production-guard visibility) → **4** manning fusion ([[line-manning-vision]], the one with real human consequences — needs an explicit conversation about what's recorded and who sees it before any camera goes up).

**Two UNKNOWNs blocking a real manager view** (logged in the register, asked in the ledger): is **FCT/rework quality data** in any readable DB? and is there a **machine-readable production plan/schedule** to compare actual output against? Without a plan there is nothing to say a good day was good enough against.

**Dependencies to confirm before Phase 0 is worth much:** `/api/health` ([[pvs-line-health]]) and downtime capture ([[pvs-downtime-capture]]) were BUILT 2026-08-18 but deployment is unconfirmed. Also: Floor Ai must be added to the NAS cutover repoint inventory ([[mcs-cutover-readiness]]) — its DB host is config, not hardcoded.
