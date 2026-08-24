---
name: line-pc-time-sync
description: All 5 line PCs now auto-sync clocks via internet NTP; L2-L5 were previously not syncing.
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-24T01:11:41.715Z
---

Set up 2026-08-24: all five line PCs (L1-L5) auto-sync their clocks to internet NTP. W32Time StartupType=Automatic, manualpeerlist = `pool.ntp.org,0x9 time.windows.com,0x9 time.google.com,0x9`, syncfromflags=manual.

**Before the fix:** L1 was already syncing (pool.ntp.org). **L2 was free-running** (W32Time running but never synced). **L3, L4, L5 had W32Time STOPPED** — not syncing at all. All corrected + confirmed syncing (stratum 2-3) that morning. Clocks happened to be correct at the time, but L2-L5 would have drifted.

Clock accuracy matters because DailyProductionCount / StockOut / ConsumedReels rows are stamped from each line PC's local clock; drift would misdate production and attrition.

Check: `w32tm /query /status` and `Get-Service W32Time`. Fix: Set-Service W32Time -StartupType Automatic; Start-Service; `w32tm /config /manualpeerlist:"..." /syncfromflags:manual /update`; `w32tm /resync /rediscover`. Related: [[harness-date-utc-offset]], [[remote-access-from-home]].
