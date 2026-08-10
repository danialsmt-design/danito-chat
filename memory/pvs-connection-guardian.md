---
name: pvs-connection-guardian
description: netguard — resident self-healing watchdog on each line PC that keeps the path to the parts DB alive; fixes the static-Wi-Fi failure mode.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-10T03:34:56.305Z
---

**netguard** = the PVS line "connection guardian". Source: `PVS/deploy/netguard.ps1`. Deployed to `C:\PvsLineApp\netguard\netguard.ps1` on the line PC, run by a **SYSTEM scheduled task `PvsNetGuard`** (AtStartup, restart-on-fail) — tokenless, laptop-independent, survives reboot.

What it does: every 20s it TCP-tests the parts DB (`192.168.0.134:1433`). While reachable it does **nothing** (fully passive — never touches a healthy link). After 3 consecutive misses (~60s) it runs an escalating ladder, re-testing after each step, stopping on success: (1) fix a **broken static config** (DHCP off + no gateway → switch adapter to automatic + renew), (2) DHCP-renew Ethernet, (3) DHCP-renew Wi-Fi, (4) re-associate Wi-Fi to a whitelisted SSID (only if `netguard.ssid` exists). 5-min cooldown anti-flap. Writes `netguard.log` + `netguard-status.json` (observable remotely / by the app).

Why it exists — the failure it fixes (Line 5, 2026-08-10): operator "couldn't scan operator ID". Root cause = the badge lookup hits the `Users` table on the central DB, and Line 5's **Wi-Fi had been set to a static IP with NO gateway** (DHCP disabled) → couldn't reach `192.168.0.134` → every scan **timed out with no on-screen error**. DB itself was fine (Line 1 proved it). NOT a "move to LAN" problem — the fix is restoring the Wi-Fi to DHCP. Guardian did it in 51s and keeps it healed.

Gotchas: the script param `-Db` collides with the `-Debug` common-param alias `db` under `[CmdletBinding()]` → binding error before any code runs, empty log, task result 0x1. Fix = **no `[CmdletBinding()]`** (already removed). Deploy note: guardian repair renews Wi-Fi → brief Tailscale blink; it heals locally regardless. Related: [[pvs-line5-deploy]], [[remote-access-from-home]]. Rollout to lines 1/2/3/4 was offered/pending as of 2026-08-10.

Known open UX gap: the scan screen should show "⚠ Cannot reach parts database" instead of a silent hang when the DB is unreachable — not yet built.
