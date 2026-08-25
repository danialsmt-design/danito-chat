---
name: whatsapp-bridge-pi
description: "The one WhatsApp channel: Raspberry Pi 'gms-wabridge' on Tailscale 100.90.248.92 — REST :8080 + MCP/SSE :8081, both live; the laptop-local MCP server still points at dead localhost:8080."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-24T00:00:00.000Z
---

The WhatsApp bridge/API lives on a **Raspberry Pi 3 "gms-wabridge"**, not the laptop:
- **Tailscale 100.90.248.92** (the way to reach it from anywhere). Also **10.1.1.145** /
  `gms-wabridge.local` on the GMSENG WiFi (10.1.1.x). SSH `gmstest@10.1.1.145`
  (password is in Danial's private connection card — deliberately NOT stored here).
- WhatsApp account **+60122185237** (self-chat), paired 2026-08-04.
- Services (systemd, auto-start): **whatsapp-bridge** = Go/whatsmeow REST API on **port 8080**
  (`POST http://100.90.248.92:8080/api/send` with `{"recipient":"60...","message":"..."}` — no auth,
  one call); **whatsapp-mcp** = Python MCP/SSE on **8081** (`http://100.90.248.92:8081/sse`).
  Logs: `journalctl -u whatsapp-bridge`. On-Pi code/DB: `/home/gmstest/whatsapp-mcp/...`.
- Text only. File attachments need the file staged on the Pi — **not yet solved**.
- **✅ DELIVERY VERIFIED END-TO-END 2026-08-24 21:20 MYT:** direct
  `POST /api/send` to 60122185237 returned `{"success":true}` (http 200, 1.05s) **and Danial confirmed
  the message arrived on his phone**. So this is not merely reachable/healthy — it is a proven
  delivering send path.

**✅ REACHABILITY (re-verified 2026-08-24):** the Pi **is on Tailscale now** — ping OK (~150ms),
:8080 answers (404 on `/`, which is just "no route at /"; `/api/send` is the endpoint), :8081/sse
holds a 200 SSE stream. This **supersedes the 2026-08-04 finding** that the Pi was 10.1.1.x-only and
unroutable from the laptop; option (b), `tailscale up` on the Pi, was taken.

**✅ SEND PATH FIXED (2026-08-24):** `whatsapp-mcp\whatsapp-mcp-server\whatsapp.py:11` used to hardcode
`http://localhost:8080/api`, which had gone dead when the bridge moved to the Pi (hence the old
`WinError 10061`). It now reads
`WHATSAPP_API_BASE_URL = os.environ.get("WHATSAPP_API_BASE_URL", "http://100.90.248.92:8080/api")`
— Pi by default, env-overridable. Backup at `whatsapp.py.bak`. All senders that go through the
`whatsapp` MCP tool (this session + all four scheduled tasks) now hit the Pi.

**🗑️ LAPTOP BRIDGE REMOVED (2026-08-24, on Danial's instruction "remove all other whatsapp bridge"):**
the laptop was a **second, duplicate pairing** with its own DB. Removed as follows —
- Startup autostart `whatsapp-bridge.vbs` (Start Menu\Programs\Startup) → renamed `.vbs.disabled`.
  **This was what kept resurrecting it**; it had run until 15:02 that day.
- Whole install `whatsapp-mcp\whatsapp-bridge\` (exe, main.go, and `store/` with `whatsapp.db` pairing
  creds + `messages.db` archive + 3 media dirs) → **moved, not deleted**, to
  `whatsapp-mcp\_REMOVED-laptop-bridge-2026-08-24\`. Restore by moving it back and re-enabling the .vbs.
- Verified afterwards: sweep of every Tailscale node (laptop, L1–L5, Pi) on :8080 — **only the Pi answers**.

**Consequence — MCP read tools are now broken by design:** `MESSAGES_DB_PATH` in `whatsapp.py` still
points at the archived laptop `store/messages.db`, which no longer exists there, so `list_chats` /
`list_messages` / `search_contacts` fail. **Sending is unaffected** (it goes to the Pi over HTTP, no DB).
To restore reads, point the `whatsapp` MCP server at the Pi's own MCP/SSE endpoint
`http://100.90.248.92:8081/sse` instead of running the local Python server — needs an app restart, and
the Desktop config currently uses stdio (`uv run main.py`), so the remote-server format needs checking first.

**⚠️ Still open — phone-side unlink:** deleting files does NOT unlink the device on WhatsApp's servers.
The old laptop session may still show under **WhatsApp → Settings → Linked Devices**. Danial must log that
entry out from his phone; nobody else can.

**Not a bug:** `gmail-whatsapp-digest\digest.py:27` defaults to `localhost:8080/api/send` — correct,
because that script is built to run ON the Pi, where localhost IS the bridge. It honours `BRIDGE_URL`
if it ever runs elsewhere.

**Channel count: ONE.** Every sender — this session's MCP tool, all four scheduled tasks, Floor Ai,
MCS Ai, the line PCs — funnels through this single Pi bridge. It has no redundancy: if the Pi is off,
Danial and Raja Rao hear nothing from anything.

Related: [[gmail-whatsapp-digest]], [[floor-ai-manager-agent]], [[mcs-ai-agent-spec]], [[remote-access-from-home]].
