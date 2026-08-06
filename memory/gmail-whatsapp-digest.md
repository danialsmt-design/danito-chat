---
name: gmail-whatsapp-digest
description: "Scheduled task that WhatsApps a 2-hourly Gmail digest to Danial's self-chat; depends on the local WhatsApp bridge being up."
metadata: 
  node_type: memory
  type: project
  originSessionId: a6f80384-4948-44dd-ad29-0de150e0907d
  modified: 2026-08-04T16:08:12.153Z
---

Local scheduled task `gmail-to-whatsapp-digest` (in `.claude/scheduled-tasks/`) runs every 2 hours (`0 */2 * * *`, local time): searches Gmail inbox for mail received in the last 2h, summarizes it, and sends the digest via the WhatsApp MCP to Danial's own number **60122185237** (the "message yourself" self-chat). Sends nothing when there's no new mail.

**Key dependency:** the WhatsApp *send* path needs the local WhatsApp bridge daemon running on `localhost:8080`. The read tools (list_chats/search_contacts) work off the local message DB even when the bridge is down, but `send_message` fails with `WinError 10061` if the bridge isn't running. Also, local scheduled tasks only fire while the Claude app is open.

**Bridge location:** `C:\Users\Lourdes Gunadasan\whatsapp-mcp\whatsapp-bridge\whatsapp-bridge.exe` — just run that exe (leave the window open); it auto-reconnects from the `store` folder (no QR re-scan) and starts the REST server on :8080. The MCP server half is at `whatsapp-mcp\whatsapp-mcp-server` (uv run main.py), wired in claude_desktop_config.json. Go is at `C:\Program Files\Go\bin\go` if a rebuild is ever needed.

**Heartbeat:** on the daily 08:00 local run, if there's no new mail it still sends a one-line "digest is running" message; all other empty runs stay silent.

**Pi migration (chosen 2026-07-25):** to run without the laptop, Danial is moving this to an always-on Raspberry Pi at home, keeping personal WhatsApp. Build files are at `C:\Users\Lourdes Gunadasan\gmail-whatsapp-digest\` (digest.py, auth_gmail.py, run.sh, whatsapp-bridge.service, SETUP.md). Architecture: Pi runs the whatsapp-bridge as a systemd service (re-link device via QR) + a cron job (`0 */2 * * *`) running digest.py, which pulls Gmail via the Gmail API (read-only OAuth token.json), summarizes with the Claude API (default model claude-haiku-4-5), and POSTs to the local bridge at localhost:8080/api/send. No MCP layer, no Claude app. Once live, disable the in-app scheduled task to avoid duplicate digests.

**Blocked in-app since at least 2026-07-30 (re-confirmed 2026-08-05):** the scheduled task's Step 1 cannot run — there is no Gmail MCP server connected to the Claude app any more (`ToolSearch "+gmail"` finds nothing; `list_connectors` for mail/email returns empty). Only Calendar, Drive, Recordings and WhatsApp connectors are present. So every in-app run now produces no digest. Either reconnect a Gmail connector, or finish the Pi migration above and delete the in-app task.

Danial's WhatsApp number: **60122185237**. (Note: a contact "AASyaffeq Danial" 60129555237 is a different person, not him.)

Related: [[schedulle-project]], [[user-danial]].
