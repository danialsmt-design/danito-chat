---
name: pvs-dpc-recording-flag
description: writeProductionCount per-line flag controls whether PVS records production to DailyProductionCount; a redeploy silently reset it and 3 lines went unrecorded for days.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-17T03:09:29.465Z
---

**Incident (2026-08-17):** Lines **1, 3, 4** were producing but writing NOTHING to `DailyProductionCount` — L1/L3 blind since 11 Aug, L4 since 14 Aug (discovered while comparing weekly output; L1 running 253 bph with last DB row 11 Aug). Root cause: the per-line config flag **`writeProductionCount`** (in `C:\PvsLineApp\line.config.json`) was **false**. When true, PVS writes each line's board count to `DailyProductionCount` (tagged `SenderIp='PVS'`) every 30 min; when false PVS records nothing AND doesn't even accumulate (SessionCoordinator.cs:650) so the gap is unrecoverable from PVS.

**Why it happened / how it recurs:** a redeploy re-copied `line.config.json` from the repo templates (`deploy/lineN.config.json`), and 4 of 5 templates still said `writeProductionCount:false` from the pilot era. So any redeploy silently reverts a line to "not recording." Line 3/4 additionally never had PVS as the writer — their counts came from an **external logger PC** (L3=192.168.0.132, L4=192.168.0.190) that died on 11/14 Aug with nothing to replace it.

**Fix applied (all writes as `pvs_ro`, which CAN insert DPC — L2/L5 do daily):** set `writeProductionCount:true` on L1/L3/L4 live configs + restarted the `PvsLineApp` scheduled task (backups `line.config.json.bak-20260817`). **All 5 lines now recording.** Repo templates `deploy/line2..5.config.json` fixed to `true` (were the L2/L5 landmines). Verifier: `scratchpad/dpc-recording-monitor.ps1`.

**Prevention (3 layers):** (1) templates fixed [done]; (2) **deploy must NOT clobber a live `line.config.json`** — it's placed manually per DEPLOYMENT-RUNBOOK §3; add a post-deploy assert that `writeProductionCount` is still true; (3) **producing-but-not-recording monitor** = MCS Ai guardian job #1 ([[mcs-ai-agent-spec]]) — alarm when bph>0 but no fresh DPC row (would've caught this in 30 min not 6 days).

**Same-pattern landmine to audit:** **`syncStockOuts`** is the same kind of per-line flag — a redeploy resetting it to false silently disables reel-retire/StockOut write-back ([[pvs-reel-retire-rank-rule]]). Check it per line the same way. **Backfill owed:** L1/L3 12–17 Aug + L4 14–17 Aug counts, recoverable only from the machine PWB counters, not PVS.
