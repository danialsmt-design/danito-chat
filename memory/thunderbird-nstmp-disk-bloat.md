---
name: thunderbird-nstmp-disk-bloat
description: "Thunderbird's failed-compaction nstmp-* files silently ate 195GB of the Dell laptop's C: drive; recurs and needs periodic cleanup."
metadata: 
  node_type: memory
  type: project
  originSessionId: 5515b171-01eb-4dc7-b525-d548afcd29d2
  modified: 2026-07-29T13:27:04.538Z
---

On the Dell Inspiron 7490 (main laptop), Thunderbird's profile at
`AppData\Roaming\Thunderbird\Profiles\jj1qjhq1.default-release\Mail\mail.shinsmt.com`
accumulated **127 orphaned `nstmp-*` files totalling 194.87 GB**, dated 24 Jul 2020 → 30 Jun 2026.
Cleaned up 29 Jul 2026: C: went from 15.2 GB free (3.3%) to 209.9 GB free (45.6%).

**Why:** `nstmp-*` are temp copies Thunderbird writes while compacting a folder. If compaction is
interrupted, the temp file is orphaned — Thunderbird never reuses or deletes it. The shinsmt.com
Inbox is ~10 GB, so each failed compaction stranded ~3.5 GB. Compaction has clearly been failing
repeatedly for years, so **this will come back**.

**How to apply:** When disk space on this machine gets tight, check for `nstmp*` first — it is the
single biggest offender and is pure garbage. Close Thunderbird, then:
`Get-ChildItem "<Mail dir>" -Recurse -File -Force | Where-Object { $_.Name -like 'nstmp*' } | Remove-Item -Force`
Only match `nstmp*` — real mail folders are extensionless files with no prefix (Inbox, Sent, Junk).
Root fix is to stop the compaction failures (the ~10 GB Inbox is the likely cause — archive it down).

Related: [[pc-health-known-issues]]
