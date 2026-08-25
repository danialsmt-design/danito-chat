---
name: whatsapp-route-via-wabridge
description: "Standing rule from Danial (2026-08-24): every WhatsApp message he asks to be sent goes out through the gms-wabridge Pi — no other path."
metadata: 
  node_type: memory
  type: feedback
---

**Rule:** any message Danial asks to be sent on WhatsApp is sent **through `gms-wabridge`**
(the Raspberry Pi, Tailscale **100.90.248.92**) — never through any other bridge, pairing, or path.

**Why:** the laptop-side bridge is a dead second pairing (stopped, nothing on laptop :8080, its DB frozen
at 2026-08-24 15:02). Anything that still aims at `localhost:8080` silently fails with `WinError 10061`.
The Pi is the only live, always-on sender, and it is what the Floor Ai and MCS Ai briefs already tell
every agent to use. One channel, deliberately — see [[whatsapp-bridge-pi]].

**How to apply:**
- Preferred, always works regardless of MCP server state — POST straight to the Pi:
  `curl -X POST http://100.90.248.92:8080/api/send -H "Content-Type: application/json" -d '{"recipient":"60...","message":"..."}'`
  (no auth, single call, text only).
- The `whatsapp` MCP tool is now also pointed at the Pi (`whatsapp.py:11`), but a **running** MCP server
  process keeps whatever URL it loaded at spawn — so after that edit, the fix only applies to sessions
  started since. When in doubt this session, use the curl above.
- **Scope: on request only.** This rule says *where* a message goes, NOT that messages may be sent
  unprompted. Still ask before sending anything on Danial's behalf, and still confirm the recipient —
  Danial **60122185237**, Raja Rao **60163327003** (reports only, and only when Danial approved that
  recipient for that report).
- Text only; attachments need staging on the Pi, still unsolved.
- **The laptop Linked Device STAYS linked** (Danial, 2026-08-24: "leave that") — but it is **never a send
  path**. Its bridge is retired at `whatsapp-mcp\_REMOVED-laptop-bridge-2026-08-24\`, autostart disabled
  and the exe renamed `.exe.disabled` so it cannot be launched by accident. Do not start it, do not
  re-enable the .vbs, and never send through that pairing. The pairing entry existing on the account is
  fine and intentional; only *sending* through it is forbidden.
- Note: WhatsApp may itself log out a linked device after a long stretch of inactivity. If that entry
  disappears from Linked Devices on its own, that is WhatsApp's doing, not a fault to chase.

Related: [[whatsapp-bridge-pi]], [[gmail-whatsapp-digest]], [[floor-ai-manager-agent]].
