---
name: pvs-line3-juki-janets
description: "Line 3 is JUKI (not Sony) — machines talk over LAN via JaNets software, not serial; LINE3PVS access + network + install state."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-06T13:01:24.761Z
---

**Line 3 = JUKI, fundamentally different from the Sony lines (1/2/5).** The mounters do NOT expose a serial "report" like Sony's C1M. They communicate over **LAN/Ethernet** to JUKI's **JaNets** line-management software, which collects production data into its own local database. The COM ports on LINE3PVS (COM1,3-6, WCH card) are a dead end — probed silent at all bauds because JUKI isn't serial.

**Accessing LINE3PVS:**
- Tailscale `100.105.64.115` (MagicDNS `line3pvs.tail14c782.ts.net`); LAN Wi-Fi `192.168.0.126`.
- Cred `line3.cred` = **`LINE3PVS\User`** (the only ENABLED admin; built-in Administrator is DISABLED). WinRM needed `LocalAccountTokenFilterPolicy=1` set on the box (custom local-admin token filtering) — that's why the first connects gave "Access is denied". The password had to be reset via `Set-LocalUser`.
- SMB (445) is open and MUCH faster than WinRM `Copy-Item -ToSession` for large files (WinRM base64-encodes → <1 MB/s).

**Machine network (configured 2026-08-06):**
- Machines are on **`10.30.29.x`**. LINE3PVS **Ethernet (Realtek PCIe GbE) → static `10.30.29.100/24`, no gateway**; Windows Firewall **disabled** (JaNets requires it off). One NIC/switch reaches all machines.
- **LINE3PVS MUST stay at `10.30.29.100`** — that is where JaNets' **mosquitto MQTT broker** listens (`:1883`), and `Juki.Iss.SmtConnection.Svc` connects to it. Moving the PC off `.100` breaks JaNets (service stuck SynSent to `.100:1883`). Don't reassign it.
- **NEVER put the PC NIC on `192.168.0.x`** — reserved for the mounter's internal bus.
- The machine is at **`10.30.29.12`** = a JUKI **RS-1** (older series). Only `.12` answered the ping sweep — other machines were off/uncabled at setup.
- Don't `Restart-NetAdapter` on the Realtek NIC remotely — it came back **Disconnected** and needed a disable/enable cycle + wait to re-link. Change the IP with Remove/New-NetIPAddress only, no adapter bounce.

**How JaNets talks to the RS-1 (verified working 2026-08-06):** the RS-1 (older series) uses **SMB**, not MQTT — machine account **`isuser` / `is_user_2008`** (created ON the machine), with **`Prg`** (program folders) and **`Interface`** shares. Connection Test opens SMB to `.12:445`. In JaNets Shopfloor Setup the hierarchy is **Floor → Line → Machine** (Machine greyed until a Line parent exists; avoid **EPU mode** — it disables Learn Conditions / IP auto-check). Drag the machine from **Unassigned** onto the Line view to assign it. Machine is **learned + connected** as of 2026-08-06. JaNets app login `administrator`/`Administrator`.

**Install (done by Danial from pen drive, 2026-08-06):** JaNets 1.24 at `C:\Juki\JaNets`; services running: `JukiSmtConnSvc`, `Juki.BigData.Manager.Svc`, Sentinel (dongle). Login `administrator`/`Administrator`. Install gotchas: **dongle OUT during install**, display **100%/96 DPI**, run as admin, main setup.exe bundles VC++/MSXML/Sentinel driver. Skip "Data Manager" (server product); Component DB into a separate folder.

**PVS integration path for JUKI (not yet built — pending live production):** candidate sources on LINE3PVS, to inspect once the RS-1 is monitoring + producing: (1) the machine's **`Interface`** SMB share (where JaNets exchanges machine data — most promising for RS-1), (2) JaNets' local DB — TraceMonitor "Production result list" = program name + produced-PWB count + time window per run (the JUKI analog of the Sony count); `Juki.BigData.Manager.Svc` is the data-collection service, (3) the **mosquitto MQTT broker** on `.100:1883` — `mosquitto_sub.exe` is at `C:\Program Files\mosquitto\`; subscribe to `#` to see topics (empty until a machine publishes; RS-1 may not use MQTT, newer JUKI machines do). The vendor External Output / JSON / GCI-MEX export needs a **separate dongle license + a manual we don't have**. See [[pvs-count-two-counters-and-cap]] for how the Sony side counts.
