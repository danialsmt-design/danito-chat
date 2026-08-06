---
name: whatsapp-bridge-pi
description: "The WhatsApp bridge runs on a Raspberry Pi (10.1.1.145) — isolated on GMSENG WiFi, NOT on Tailscale, so unreachable from the laptop."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-04T15:39:07.130Z
---

The WhatsApp bridge/API lives on a **Raspberry Pi 3 "gms-wabridge"**, not this laptop:
- **10.1.1.145** on the **GMSENG WiFi (10.1.1.x)**; also `gms-wabridge.local`. SSH `gmstest@10.1.1.145`
  (password is in Danial's private connection card — deliberately NOT stored here).
- WhatsApp account **+60122185237** (self-chat), paired 2026-08-04.
- Services (systemd, auto-start): **whatsapp-bridge** = Go/whatsmeow REST API on **port 8080**
  (`POST http://10.1.1.145:8080/api/send` with `{"recipient":"60...","message":"..."}` — no auth, one call);
  **whatsapp-mcp** = Python MCP/SSE on **8081** (`http://10.1.1.145:8081/sse`). Logs: `journalctl -u whatsapp-bridge`.
- On-Pi code/DB: `/home/gmstest/whatsapp-mcp/...`.

**🔴 REACHABILITY (verified 2026-08-04):** the Pi is on 10.1.1.x and **NOT on Tailscale**. The laptop
(192.168.0.x / Tailscale 100.67.55.87) and every Tailscale machine (line1/2/5, parts PC — all 192.168.0.x)
**time out to 10.1.1.145** — the two networks don't route. So the `whatsapp` MCP tool (which hits
`localhost:8080` via a now-dead tunnel) fails with connection-refused whenever that tunnel is down.
**To send:** either (a) join the laptop to the **GMSENG WiFi** — Tailscale still reaches the line PCs, and
10.1.1.145 becomes directly reachable for the REST `/api/send`; or (b) **put the Pi on Tailscale**
(`tailscale up` on the Pi) — the permanent fix, then it's reachable from anywhere like the line PCs. See
[[gmail-whatsapp-digest]].
