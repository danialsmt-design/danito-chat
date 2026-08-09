---
name: rfid-app-pn532-tool
description: The standalone PN532 RFID reader/writer web app (dump/crack/write/clone) and the pending UHF-reader integration.
metadata: 
  node_type: memory
  type: project
  originSessionId: 5b02837f-661a-4d9b-95be-190fb8781262
  modified: 2026-08-09T21:17:15.590Z
---

Standalone RFID tool at `C:\Users\Lourdes Gunadasan\Downloads\chatgpt-ruflo\exports\rfid-app` (Node/TypeScript, ChatGPT-generated then heavily extended). Run: `npm run rfid`, open **http://127.0.0.1:4173/dump.html**. Server = `src/rfid/server.js` on port 4173.

**Hardware:** PN532 NFC reader on a CH340 (COM4, VID 1A86/PID 7523). Danial's test card `52991FE3` is a **gen2/CUID magic MIFARE Classic 1K** (block 0 UID rewritable). The old HID demo reader (VID_FFFF/PID_0035) can't read UIDs — see [[nfc-hid-reader-protocol]].

**Features I built on `/dump.html` (all tested except mfcuk):** full unredacted dump (all 64 blocks incl. block 0 + sector-trailer Key A/access/Key B); Read / Crack keys (dictionary ~55 keys) / Clear / Save / Load(json); Write block+verify; Clone block 0 (gen1a backdoor + gen2 keyed write); **Full clone (all 64 blocks)**; Deep crack via bundled `mfcuk.exe` (darkside — experimental; `mfoc.exe` is BROKEN/segfaults; libnfc tools can WEDGE COM4 → replug).
Endpoints: `/api/pn532/{dump,write,clone,clone-card,deepcrack,read-block,read-all}`. Key files: `pn532-reader.ts`, `server.ts`, `domain.ts`, `public/dump.html`.
libnfc crack tools live in `C:\Users\Lourdes Gunadasan\Desktop\NfcKing\nfc-bin` (mfcuk works, mfoc broken); libnfc.conf connstring = pn532_uart:COM4:115200.

**PENDING (next session): integrate a UHF RFID reader/writer** Danial ordered (Pinduoduo, ~RM105, ISO18000-6C/EPC Gen2, 902–928MHz, USB HID virtual-keyboard, 2dBi built-in = short range, same S-logo vendor family as the demo NFC reader). Plan: add a UHF reader to the app's reader-registry + a UHF read-EPC/write-EPC/clone section. Reverse its HID protocol like the NFC reader (proxy usb.dll shim) if HID-keyboard mode isn't enough. UID equivalent = EPC (rewritable on standard tags); TID is factory-locked (need "magic" changeable-TID tags for a full clone). Malaysia UHF band 919–923 MHz.
