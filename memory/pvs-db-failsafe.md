---
name: pvs-db-failsafe
description: "PVS DB resilience — fail-over second link, local hold, and a WhatsApp alert when neither DB route is reachable."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-24T14:17:07.682Z
---

Built + deployed to all 5 lines 2026-08-24 so a DB-path fault can neither stall nor lose a production count. Three layers:

1. **Fail-over second link (`SqlReelPartRepository.OpenAsync`):** tries the preferred DB link, on a connect failure falls over to a second and STICKS to whichever answered. Config `central.server` = primary (intranet `192.168.0.134,1433`), `central.fallbackServer` = a SECOND route to the SAME DB (the box's Tailscale IP `100.91.120.113,1433`). A same-IP intranet fallback is pointless (Windows picks the NIC) — the fallback must be a different address = different route. Connect Timeout dropped 15s→8s for fast fail-over. `fallbackServer` is in each line's `line.config.json` (added after the `"server"` line).

2. **Hold-and-write (already live):** if BOTH routes are down, boards stay in `dpc-state.json` and write when either returns, dated to the real day. See [[pvs-dpc-recording-flag]].

3. **DB-down WhatsApp alert (`DbHealthService`):** the 45s heartbeat pings via `PingAsync`, which now tries BOTH routes — so a false ping = nothing can reach the DB. After ~3 consecutive misses (~1.5 min) it WhatsApps Danial once ("CANNOT reach the production DB on EITHER route… production is HELD…"), and once on recovery. Sender = `WhatsAppSender` → `POST http://100.90.248.92:8080/api/send {recipient,message}` (gms-wabridge Pi, on Tailscale = a different route than the intranet DB, so the alert gets out even when the DB path is down). Config `alerts.whatsAppBridgeUrl` / `alerts.whatsAppRecipient` (`60122185237`) / `alerts.dbDownAlert` — DEFAULTED in code (`AlertConfig`) so it works with no per-line config. All 5 lines can reach the bridge:8080 over Tailscale (verified). Bridge send verified HTTP 200.

**Complementary, NOT yet built:** netguard OS-level route-failover so the intranet **WiFi NIC** is used as a second path to `192.168.0.134` (the lines are dual-homed Ethernet+WiFi to that subnet). Also the NAS watchdog. Related: [[remote-access-from-home]], [[whatsapp-bridge-pi]], [[whatsapp-route-via-wabridge]], [[nas-reelpart-db-host]].
