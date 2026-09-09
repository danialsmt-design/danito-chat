---
name: nas-daiya-line-monitor
description: NAS line monitor (nas-daiya :8899) polls line PCs by LAN IP from env PVS_LINES; those are DHCP Ethernet leases and drift — how to diagnose and fix (2026-09-09 L1/L3 outage).
metadata: 
  node_type: memory
  type: project
  originSessionId: 31b602de-df34-4c69-a1c2-678276a481f3
  modified: 2026-09-09T03:15:52.080Z
---

`nas-daiya` (Synology gmssmt1, 192.168.0.169 / Tailscale 100.125.22.119, project `/volume1/docker/nas-daiya`) polls each line's PVS :5199 over the **factory LAN** using `PVS_LINES` in its docker-compose. Feeds `/api/all`, which the MCS schedule page reads (`LINE_MONITOR_URL`).

Line PCs sit on **Ethernet with DHCP** (Wi-Fi adapters are down, their static Wi-Fi IPs no longer answer). As of 2026-09-09: L1 .165, L2 .162, L3 .200, L4 .144, L5 .190. Each PC needs the inbound rule `PVS web UI 5199 (NAS monitor)` (TCP 5199, remote 192.168.0.169 only); all five have it now.

**Why:** On 2026-09-09 `/api/all` showed L1 and L3 `ok:false` because the compose still had their old Wi-Fi IPs (.105/.126) and L1 lacked the scoped rule. Leases can move again.

**How to apply:** When a line shows `ok:false` on the NAS card, first `Invoke-Command` over Tailscale (`line<N>.cred`) with `Get-NetIPAddress` to read the live Ethernet IP and check the 5199 rule; then edit `PVS_LINES` on the NAS (Posh-SSH + `nas.cred`, `-TimeOut` not `-TimeoutSec`), `sudo docker compose up -d` in the nas-daiya folder only (never touches mcs-web / reelpart-sql), and mirror the edit into `Documents/Dantec/PVS/nas-daiya/docker-compose.yml`. Durable fix still pending: DHCP reservations per line-PC MAC on the factory router. Related: [[pvs-deploy-when-line-idle]], [[mcs-line-schedule]], [[pvs-no-smb-use-remoting]].
