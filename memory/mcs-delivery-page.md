---
name: mcs-delivery-page
description: MCS delivery page + daily WhatsApp reminders (D.O./invoice to Muli, packaging to Surianti/Danish/Masngot) — built 2026-09-10; NAS→Pi fixed 2026-09-10 by enabling TUN on Synology Tailscale.
metadata:
  type: project
---

**Built 2026-09-10** (Danial's ask): `delivery.html` + `/api/delivery/*` in the MCS app
(`app/Delivery/{DeliveryModels,DeliveryData,DeliveryNotifier,DeliveryEndpoints}.cs`). Lists Canon POs
to ship TODAY and the NEXT day (tomorrow, Sunday skipped) with SMT status (both sides complete by
DPC counts = ✓, else "A x/y · B x/y"), overdue-undelivered, and later-this-week.

**Daily reminders (WhatsApp via gms-wabridge, POST /api/send):**
- D.O. + invoice summary → **Muli 60139233878**
- packaging reminder → **Surianti 60178212825, Danish 60122445237, Masngot 60198418367**
- send time 07:45 Mon–Sat (3-h catch-up window), once per day (LastSentDate claimed before sending);
  settings + log in SchedulePlan keys "delivery" / "delivery-log"; "Send now" / "test to my number" on the page.
- Shipped with Enabled = false; **ENABLED 2026-09-10 10:47** after Danial confirmed the test messages
  arrived ("good"). First automatic send = 2026-09-11 07:45. A redeploy inside the 07:45–10:45 window
  cannot double-send: LastSentDate is claimed before sending.

**NAS→Pi path FIXED 2026-09-10 10:43** (Danial ran it himself over SSH; the harness blocks Claude from
NAS system changes): `sudo /var/packages/Tailscale/target/bin/tailscale configure-host` then
`sudo /usr/syno/bin/synopkg restart Tailscale` — Synology Tailscale was userspace-networking (no TUN),
so the NAS could accept tailnet connections but not open them (curl 100.90.248.92 → 000). After TUN:
host → Pi answers, and the MCS container's test send to 60122185237 returned success. If the package is
ever reinstalled/updated, check `ps -ef | grep tailscaled` for `--tun=userspace-networking` and redo it.

**Why:** Muli needs the D.O./invoice list before the truck; packaging needs the same list a day ahead.
**How to apply:** never point any other sender at another bridge ([[whatsapp-route-via-wabridge]]);
related [[mcs-line-schedule]], [[whatsapp-bridge-pi]], [[frequent-contacts]].
