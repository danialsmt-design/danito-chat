---
name: mcs-ai-agent-spec
description: "Spec for the 24/7 MCS Ai agent (headless Claude on the NAS) — Danial's defining requirements. NOT built yet."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-14T12:46:34.046Z
---

The **MCS Ai agent** = a 24/7 headless Claude (Claude Agent SDK) running on the **NAS** (now unblocked: 6GB RAM as of ~2026-08-14, reachable via LAN 192.168.0.169, SSH user GLOBALSMT, DSM up). This is the "keep you alive when the laptop is off" goal. **Status: NOT built** — it's a real deployment project (Synology Docker container + Agent SDK + custom tools + service). Build it in the spun-off **"MCS Ai / NAS setup"** session, methodically. Danial has provided an Anthropic API key + credit (key must live on the NAS as an env var, NEVER in chat — the pasted ones are exposed, rotate before go-live) — see [[mcs-material-control-system]], [[nas-reelpart-db-host]].

**Guardrails (SafetyEnvelope):** autonomous = read / monitor / report / propose ONLY. **Every DB write, zero-out, or WhatsApp broadcast requires Danial's explicit approval** (agent sends the preview, waits for "go"). Never self-expands scope. Follows [[pvs-rules-bible]].

**Token-prudent (Danial's explicit requirement):** event-driven not constant polling; cheapest model that fits (Haiku for routine); tiny context (just the query result, not histories); hard console spend cap + SDK budget limit; batch work into few runs. Health probes are plain TCP/HTTP — NO LLM tokens; the model only wakes on a failure to diagnose.

**Defined responsibilities (Danial, growing list):**
1. **Line-app + DB connection guardian** (2026-08-14): poll each line's PVS `/api/status`; flag lines that are unreachable / app down. Verify each line app reaches the DB (192.168.0.134); catch burst-loss stalls + "wider query unavailable". **Manage/heal the connection** — coordinate with per-line **netguard** ([[pvs-connection-guardian]]), escalate what it can't fix, alert Danial for hands-on outages (e.g. Line 4's 15h drop).
2. **Stock-accuracy background analyses** (Danial 2026-08-14: "the kind of background analysis that improves stock accuracy") — run continuously, flag only anomalies: (a) consumption-vs-production per part/line (pieces drawn vs boards×per-board; loss beyond model-change attrition = miscount/waste signal); (b) attrition classified as model-change / run-out / early-swap (like VE4-2850-104 = pulled at L311→L309 model change, ~2% of consumption — structural, not waste); (c) stale-stock watch (reels remaining whose issue-lot completed → propose write-off); (d) excess-reels-per-part; (e) count-continuity across same-model lot changes (reels are NOT pulled on a same-model lot change — count keeps decrementing; only a MODEL change pulls reels); (f) data-quality (cross-line dup UIDs, blank-line reels, date-format traps — DailyProductionCount uses yyyy-MM-dd, other tables dd-MM-yyyy). These are SQL+arithmetic (near-zero tokens); the LLM only wakes to summarize an anomaly or draft an approval request. See [[stockout-stale-reconciliation]], [[pvs-reel-retire-rank-rule]].
3. Reports (attrition to Raja Rao, etc.).

**Reach from the NAS/agent:** DB via LAN (192.168.0.134); Pi WhatsApp bridge **100.90.248.92:8080** (`POST /api/send {recipient,message}` — all WhatsApp routes through gms-wabridge now, see [[gmail-whatsapp-digest]]); PVS line apps on `:5199` over Tailscale/LAN.
