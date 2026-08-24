---
name: harness-date-utc-offset
description: "Why Claude's \"today's date\" can read one day behind Danial's wall clock (UTC vs MYT)."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-24T01:11:30.254Z
---

Claude Code's injected "Today's date" is computed by the harness in **UTC**, not Malaysia local time (MYT = UTC+8). During the early-morning MYT hours the UTC date still shows the previous day, so Claude's context can read e.g. 2026-08-23 while the actual floor/PC clock reads 2026-08-24 (Monday).

Seen 2026-08-24 ~08:43 MYT: harness said "2026-08-23", every PC (incl. this dev PC) correctly read 2026-08-24. It is NOT a wrong clock — verify against the machine's own `Get-Date` before querying DB by date. When querying DailyProductionCount / attrition "for today", trust the user's stated day / the line PC clock, not the harness date.

Related: [[line-pc-time-sync]].
