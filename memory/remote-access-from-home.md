---
name: remote-access-from-home
description: How to reach the factory PCs from home via Tailscale (double-hop through LINE1PVS).
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-07-31T15:25:45.272Z
---

From home (off the factory LAN), the factory `192.168.0.x` network is only reachable via **Tailscale**.

**⚡ 2026-08-12 — the NAS (`gmssmt1` = 100.125.22.119) is now ON the tailnet directly** (Synology Tailscale pkg), so I reach it from the laptop without any line-PC jump host. See [[nas-reelpart-db-host]] for the plink pattern. Line 2/5 also carry `plink` for the LAN→NAS hop when needed. ⚠️ Line PCs go offline on the tailnet when powered down at night.

**⚡ UPDATED 2026-07-29 — new tailnet `danialsmt@gmail.com`, all 3 PCs on it, PVS DB now runs OVER Tailscale.**
- Tailnet `danialsmt@gmail.com`. Devices: `lourdes-gunadasan` (Danial's PC) = **100.67.55.87**; `desktop-technical` (Parts Control PC / DESKTOP-TECHNIC, 192.168.0.134) = **100.91.120.113**; `LINE1PVS` (Line PC, 192.168.0.141) = **100.69.81.105**. (The old `100.108.0.118` was a DIFFERENT account — re-login moved LINE1PVS onto danialsmt.)
- **THE BIG FIX:** the Line PC ↔ Parts PC **LAN link is broken** — the Line PC's cheap **USB Wi-Fi dongle "AIC8800D80"** drops packet bursts, so any multi-row DB read stalls 40s ("semaphore timeout"); small queries/pings are fine. Proven: burst-ping Line→Parts = 100% loss over LAN but **0% over Tailscale**; the stalling ProductBOM query = 40s LAN vs **0.1s over Tailscale**. So **PVS's DB connection was switched to Tailscale**: `line.config.json` → `central.server = "100.91.120.113,1433"` (Parts PC Tailscale IP + port, NOT the `DESKTOP-TECHNIC\SQLEXPRESS` named instance — MagicDNS/SQL-Browser is unreliable). Report, live inventory, StockOuts sync all work again. Config backed up in `C:\PVS_Backups\`. **Long-term: wire the Line PC via ethernet** — Tailscale still rides the dongle for internet, so if the dongle dies entirely Tailscale stops too.
- Tailscale CLI is per-user on Windows: from a WinRM session it 401s ("in use by LINE1PVS\User" / "DESKTOP-TECHNIC\ACER-PC"). To (re)login a PC, drop a `CONNECT-TAILSCALE.bat` (`"C:\Program Files\Tailscale\tailscale.exe" login`) on that PC's desktop and have the interactive user run it + approve in a browser. Read a peer's Tailscale IP with `Get-NetIPAddress ... InterfaceAlias -match 'Tailscale'` (no CLI needed).

**On the shop LAN (Danial's PC 192.168.0.179, or when home Tailscale is down):** reach the Line PC at **192.168.0.141** (`Resolve-DnsName LINE1PVS`) and the Parts PC at **192.168.0.134** directly over WinRM/SQL — Danial's PC has a GOOD link to the Parts PC (0.2s queries) even though the Line PC doesn't. SQL as `pvs_ro` over `192.168.0.134,1433` (pwd in `C:\PvsLineApp\db-password.txt`) works from Danial's PC when the WinRM double-hop is down.

**(historical, pre-2026-07-29):** Only LINE1PVS was on the tailnet (`100.108.0.118`), Parts PC reached by double-hop through it.

**Home laptop prereqs (already done 2026-07-25):** client `TrustedHosts = 192.168.0.141,192.168.0.134,100.*` (set via elevated PowerShell — needs admin, not settable from this non-elevated session). Creds: `~/line1.cred` (`LINE1PVS\linesvc`) and `~/partsctl.cred` (`DESKTOP-TECHNIC\dbsvc`). LINE1PVS's own `TrustedHosts` set to `192.168.0.134,192.168.0.141,100.*` for the onward hop.

**Reach LINE1PVS directly:** `Invoke-Command -ComputerName 100.69.81.105 -Credential (Import-CliXml ~/line1.cred) { ... }`.

**✅ USE MAGICDNS NAMES, NOT IPs (verified 2026-07-31).** MagicDNS is ON; tailnet suffix
**`tail14c782.ts.net`**. `line1pvs` resolves (CNAME → `line1pvs.tail14c782.ts.net` → 100.69.81.105) and
`http://line1pvs:5199` returns 200. Names follow the device, so prefer `line1pvs` /
`desktop-technical` everywhere. ⚠️ **Tailscale IPs do NOT drift** — a node's 100.x address is stable
for that node's life; the old `100.108.0.118` changed only because the PC was re-logged into a
DIFFERENT account, creating a new node. So there is nothing to "pin". (I initially told Danial to pin
the IP — wrong advice, corrected.)
**⚠️ DO disable KEY EXPIRY** in the Tailscale admin console (Machines → each of `line1pvs`,
`desktop-technical`, Danial's PC). Default keys expire in 180 days and the device then silently drops
off the tailnet until someone logs in INTERACTIVELY — on an unattended shop-floor PC that kills remote
access and needs a person physically at the machine.
**Still on raw IP:** PVS `central.server = "100.91.120.113,1433"` — could become
`desktop-technical,1433` (keep the explicit port; the earlier "MagicDNS unreliable" note was about SQL
Browser resolving the NAMED INSTANCE, not MagicDNS itself). Not yet changed.

**🌐 PVS WEB UI NOW REACHABLE FROM HOME (2026-07-31).** Was loopback-only. Changed the `PvsLineApp`
scheduled-task argument `--urls "http://localhost:5199"` → `"http://0.0.0.0:5199"` and added firewall
rule **"PVS web UI 5199 (Tailscale only)"** (inbound TCP 5199, **RemoteAddress `100.64.0.0/10`**) so
the tailnet can reach it but the **factory LAN cannot**. Line PC's own browser is unaffected
(loopback still works). Browse **http://line1pvs:5199/** — `index.html` (monitor), `verify.html`
(operator), `setup.html` (model/lot), `top5.html` (kiosk), `report.html` (daily). To undo: restore the
localhost argument and delete the rule.

**Reach the Parts Control PC directly — NO double hop needed** (verified 2026-07-30):
```
Invoke-Command -ComputerName 100.91.120.113 -Credential (Import-CliXml ~/partsctl.cred) { ... }
```
`dbsvc` **is a local administrator** on that box (even though it is read-only inside SQL), so OS-level
checks, services and event logs are all readable in one hop. Ports open there: 80 (IIS), 135, 445,
1433, 5985. **RDP (3389) is closed.** Query SQL in-session via `sqlcmd -S ".\SQLEXPRESS" -E`.

Caveat: that PC has only 4 GB RAM and sits at ~87% used, so it is slow — **recursive filesystem
scans over WinRM time out.** Keep remote commands targeted (per-folder sizes, not whole-drive walks).

See [[parts-control-db-write-access]], [[parts-control-pc-health]] and [[sony-smt-parts-verification]].
