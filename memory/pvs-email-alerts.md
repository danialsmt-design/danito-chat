---
name: pvs-email-alerts
description: PVS sends alert email from pvsbangi@gmail.com via SMTP (app password in ~/pvsmail.cred) — the reliable channel since the WhatsApp Pi bridge is flaky.
metadata: 
  node_type: memory
  type: reference
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-05T08:23:48.010Z
---

**PVS alert email channel** (set up 2026-08-05, chosen because the WhatsApp bridge on the Pi is
unreachable/flaky — see [[whatsapp-bridge-pi]]):
- **Sender:** `pvsbangi@gmail.com` ("PVS Bangi" — a dedicated Gmail for PVS alerts).
- **Send path:** SMTP `smtp.gmail.com:587`, `EnableSsl=$true`, auth = the account email + a Gmail **App
  Password** (2-Step Verification on the account). The app password is stored DPAPI-encrypted in
  **`~/pvsmail.cred`** on `lourdes-gunadasan` (created via `Get-Credential -UserName 'pvsbangi@gmail.com' |
  Export-CliXml ~/pvsmail.cred`). Revocable in the Google account's App Passwords page.
- **Verified working** 2026-08-05: `System.Net.Mail.SmtpClient` sent a self-test successfully (pw length 19 —
  the 16-char app password pasted with its spaces; Gmail accepts it).
- **Purpose:** the **machine cycle-time monitor** — alert when a machine's own `ET` cycle time (from C1M, see
  [[pvs-c1m-machine-counter]]) rises **>10% above baseline for 2 consecutive reads**, on Line 1 + Line 2.
  Runs on the laptop (keeps mail creds off the line PCs).

**LINE 1 alert team (roles → email, all @shinsmt.com; given by Danial 2026-08-05):**
- **Raja Rao** — `raja.rao@shinsmt.com` — **Production Leader (overall)**
- **Danish** — `danish@shinsmt.com` — **Parts Control**
- **Kalsom** (Mas Kalsom) — `maskalsom@shinsmt.com` — **Production**
- **Suriyanti** — `surianti@shinsmt.com` — **QC**
- **Badarul** — `badarul@shinsmt.com` — **Planning**
- **Masngot** — `masngot@shinsmt.com` — role TBD
These are role-based so alerts can route by type (cycle-time/machine → production leader + production; parts
shortage → parts control; quality → QC; schedule/lot → planning). Use these @shinsmt.com addresses, NOT the
older contact-card ones. Danial's own email for "and me" still TBD (gmail vs icloud).
