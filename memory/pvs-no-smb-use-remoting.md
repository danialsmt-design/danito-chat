---
name: pvs-no-smb-use-remoting
description: "Danial's rule (2026-09-07): never push files to line PCs over SMB admin shares; use direct access (WinRM/PowerShell remoting over Tailscale or LAN) instead."
metadata:
  type: feedback
---

**Rule:** do NOT copy files to the line PCs via SMB (`\<ip>\C$\...`, `New-PSDrive` to the admin share). Danial: "don't use SMB, we have direct access through Tailscale or LAN." Use WinRM / PowerShell remoting (`New-PSSession` + `Invoke-Command`, `Copy-Item -ToSession`) — the same path the config edits and health checks already use.

**Why:** he wants one access path to the boxes (the remoting sessions already authorised per line), not admin-share file drops that bypass it.

**How to apply:** for config edits, run the edit inside `Invoke-Command` on the box (as done for L1) or `Copy-Item -ToSession`. Note `deploy/deploy-pvs.ps1` still copies the publish set over SMB internally — if Danial wants that changed too, switch it to `Copy-Item -ToSession`; ask before rewriting the sanctioned deploy guard. Context: 2026-09-07 rollout of the robot build to L2–L5 ([[reel-delivery-robot]], [[pvs-deploy-when-idle]]).
