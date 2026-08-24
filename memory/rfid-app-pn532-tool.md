---
name: rfid-app-pn532-tool
description: The standalone PN532 RFID reader/writer web app (dump/crack/write/clone) and the pending UHF-reader integration.
metadata: 
  node_type: memory
  type: project
  originSessionId: 5b02837f-661a-4d9b-95be-190fb8781262
  modified: 2026-08-23T14:26:12.736Z
---

Standalone RFID tool at `C:\Users\Lourdes Gunadasan\Downloads\chatgpt-ruflo\exports\rfid-app` (Node/TypeScript, ChatGPT-generated then heavily extended). Run: `npm run rfid`, open **http://127.0.0.1:4173/dump.html**. Server = `src/rfid/server.js` on port 4173.

**Hardware:** PN532 NFC reader on a CH340 (COM4, VID 1A86/PID 7523). Danial's test card `52991FE3` is a **gen2/CUID magic MIFARE Classic 1K** (block 0 UID rewritable). The old HID demo reader (VID_FFFF/PID_0035) can't read UIDs — see [[nfc-hid-reader-protocol]].

**Features I built on `/dump.html` (all tested except mfcuk):** full unredacted dump (all 64 blocks incl. block 0 + sector-trailer Key A/access/Key B); Read / Crack keys (dictionary ~55 keys) / Clear / Save / Load(json); Write block+verify; Clone block 0 (gen1a backdoor + gen2 keyed write); **Full clone (all 64 blocks)**; Deep crack via bundled `mfcuk.exe` (darkside — experimental; `mfoc.exe` is BROKEN/segfaults; libnfc tools can WEDGE COM4 → replug).
Endpoints: `/api/pn532/{dump,write,clone,clone-card,deepcrack,read-block,read-all}`. Key files: `pn532-reader.ts`, `server.ts`, `domain.ts`, `public/dump.html`.
libnfc crack tools live in `C:\Users\Lourdes Gunadasan\Desktop\NfcKing\nfc-bin` (mfcuk works, mfoc broken); libnfc.conf connstring = pn532_uart:COM4:115200.

**UHF reader — INTEGRATED (2026-08-23).** MagicRF M100/MT6 chip, VID_FFFF/PID_0035 over the generalHID transport (NOT keyboard mode). Driver = `uhf.py` in the app root (proven: `python uhf.py read` → `{"ok":true,"epc":...}`, `write <epc>` reads back). Protocol: inner frame `BB <type> <cmd> <PL16BE> <params> <sum&0xFF> 7E`, wrapped in the AA/xor/BB HID frame, report id 0x01 out / 0x03 in, 256-byte feature reports. read=cmd 0x22, write=0x49 (MUST Set-Select 0x0C to current EPC first or write is rejected). Server shells out via `runUhf()` → routes `POST /api/uhf/read` and `POST /api/uhf/write`. UI: "UHF (915 MHz) tag — EPC" panel at top of dump.html — Read EPC / Use as write value / Write EPC; clone = read source EPC → use as write value → swap tag → write. UID equivalent = EPC (rewritable); TID factory-locked (need changeable-TID magic tags for a true full clone). Malaysia UHF band 919–923 MHz.
