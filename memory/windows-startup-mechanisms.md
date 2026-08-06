---
name: windows-startup-mechanisms
description: Trimming Windows startup needs four separate mechanisms checked; Task Manager and the Run keys alone miss two of them.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5515b171-01eb-4dc7-b525-d548afcd29d2
  modified: 2026-07-29T14:02:52.334Z
---

Auditing what launches at boot on Windows 11 requires checking **four** places. Checking only the
Run keys (or only Task Manager) gives a false "it's disabled" result — learned the hard way on
29 Jul 2026 when Teams and WPS still appeared in the tray after their Run entries were disabled.

1. **Registry Run keys** — `HKCU\...\Run`, `HKLM\...\Run`, `HKLM\SOFTWARE\WOW6432Node\...\Run`.
   Enabled/disabled state lives separately in `...\Explorer\StartupApproved\Run` (and `Run32`).
   First byte: **2 or 6 = enabled, 1 or 3 = disabled.** Treating 1 as enabled gives a wrong count.
2. **Startup folders** — per-user `shell:startup` and all-users `shell:common startup`.
   Disable by renaming the `.lnk` to `.lnk.disabled`.
3. **Scheduled tasks** with logon/boot triggers, `TaskPath = '\'`. Note some apps (WPS/Kingsoft)
   **re-enable their own tasks**, so re-check after a reboot.
4. **Packaged (Store/Appx) startup tasks** — the one everybody misses. Under
   `HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\CurrentVersion\AppModel\SystemAppData\<PackageFamilyName>\<TaskName>`,
   value `State`: **2 = enabled, 1 = disabled (by user), 0 = off.** This is how Windows 11 Teams
   autostarts (`MSTeams_8wekyb3d8bbwe\TeamsTfwStartupTask`) — its Run-key entry is a decoy.

Related: [[pc-health-known-issues]]
