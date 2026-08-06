---
name: pc-health-known-issues
description: "Main laptop is a Dell Inspiron 7490 (i7-10510U, 16GB, 512GB SSD); open issues are a Dell ServiceShell memory leak and a run of BSODs."
metadata: 
  node_type: memory
  type: project
  originSessionId: 5515b171-01eb-4dc7-b525-d548afcd29d2
  modified: 2026-07-29T14:02:42.541Z
---

Main machine: **Dell Inspiron 7490** — i7-10510U (4c/8t), 16 GB (2×8 SK Hynix @2133), 512 GB SK hynix
BC511 NVMe. Windows 11 Pro build 26200. SSD reports Healthy, no WHEA errors — hardware is sound.

Open issues as of 29 Jul 2026:

1. **Dell ServiceShell.exe memory leak** — `C:\Program Files (x86)\Dell\UpdateService\ServiceShell.exe`
   (service `DellClientManagementService`) should be ~50 MB. Measured 29 Jul 2026: **106 MB at boot →
   3,393 MB eleven minutes later.** It is not a slow leak — it balloons in minutes, so restarting the
   service is only a momentary fix. The durable fix is disabling `DellClientManagementService` or
   uninstalling Dell Update. Single largest RAM consumer on the machine (~85% RAM used).

2. **BSOD run** — three crashes in five weeks: 20 Jun 2026 `0xEF`, 22 Jul 2026 `0x139`,
   29 Jul 2026 `0xEF`. All happened while C: was ~97% full, and `C:\Windows\Minidump` was empty
   because Windows couldn't write the dumps. Disk was freed 29 Jul — **watch whether crashes stop**.
   If they continue with free space available, it's a real fault, not starvation.

3. Minor: Wi-Fi Direct Virtual Adapter fatal power-transition errors (flaky Wi-Fi after sleep);
   DNS name-resolution policy table corruption (likely NordVPN/OpenVPN residue, both autostart);
   ~30 startup programs; McAfee WebAdvisor service failing to start (leftover from removed McAfee).

Disk-space emergencies on this machine: check [[thunderbird-nstmp-disk-bloat]] first.
