---
name: mcs-db-sync-status
description: "DB sync" block on the MCS main page (go.html) + /api/sync — last restore of the NAS copy from the Parts Control PC (msdb.restorehistory); found the mirror stale since 2026-09-09 19:50 because 192.168.0.134 vanished from the LAN.
metadata:
  type: project
---

**Built 2026-09-11** (Danial: "add NAS mcs db sync to parts control pc on MCS main page").
`app/Sync/SyncEndpoints.cs` → `GET /api/sync`; block at the top of `go.html`, refreshed every minute.

How it knows: the NAS mirror is refreshed by `refresh.sh` (cron every 10 min 07:00–19:00, Mon–Sat)
= BACKUP on the Parts Control PC (192.168.0.134) → copy → RESTORE on the NAS. Every RESTORE lands in
`msdb.dbo.restorehistory` (readable by partcontrol_user), so **last restore = last sync** — no file
access needed. Newest DailyProductionCount row = data freshness. States: fresh (≤15 min) / late (≤60) /
paused (outside the window) / stale (red, pulsing, "every MCS page is working from old data").

**Incident 2026-09-11 (found by this block):** last good refresh 2026-09-09 19:50; every 10-min run from
2026-09-10 07:00 to 09-11 10:30 failed "Login timeout expired" against 192.168.0.134. Facts established:
- The Parts Control PC (DESKTOP-TECHNIC) **rebooted 2026-09-10 03:29**. It has THREE addresses:
  **Wi-Fi 192.168.0.134 (static)**, **Ethernet 192.168.0.161 (DHCP)**, **Tailscale 100.91.120.113**.
  SQL listens on all; firewall allows 1433 from Any; no block rules.
- The line PCs kept working: `line.config.json` has `central.server=192.168.0.134` + `fallbackServer=
  100.91.120.113` (Tailscale) — so PVS never noticed. Line 2 (wired) reached both LAN addresses fine.
- From the NAS the .134 (Wi-Fi) path was intermittently dead after that reboot (ARP INCOMPLETE at one
  point; "Login timeout" every run), then at 10:52 SQL logins from the NAS container to .134, .161 AND
  100.91.120.113 all succeeded again — root cause on the PC's Wi-Fi side not pinned down (power-save /
  AP), not anything done on the NAS (first failure 07:00, the NAS TUN change was 10:43).
- **busybox `nc -z` on the NAS is UNRELIABLE (reports "closed" on ports that sqlcmd connects to)** —
  probe with `docker exec reelpart-sql sqlcmd` or `/dev/tcp`, never with nc.
- refresh.sh is hard-wired to .134 (my 08-17 design). Durable fix (Danial must apply — the harness
  blocks Claude from editing that root script): try 192.168.0.134, else 100.91.120.113 (SRC variable
  for sqlcmd -S and the CIFS mount), or simply use the Tailscale address (the NAS has TUN since 09-10).
- Danial's reaction: "it was working fine all this while, why now" — answer: the PC's reboot changed
  its network behaviour; the refresh had a single hard-wired address and no fallback.

Related: [[nas-reelpart-db-host]] (refresh.sh, cron), [[parts-control-pc-health]], [[mcs-line-schedule]],
[[mcs-delivery-page]], [[nas-daiya-line-monitor]] (same DHCP-drift pattern on line PCs).
