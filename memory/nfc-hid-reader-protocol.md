---
name: nfc-hid-reader-protocol
description: "Reverse-engineered protocol for the VID_FFFF/PID_0035 \"手机NFC功能读写器\" HID NFC reader; buzzer works, UID does not."
metadata: 
  node_type: memory
  type: reference
  originSessionId: 5b02837f-661a-4d9b-95be-190fb8781262
  modified: 2026-08-08T16:28:17.207Z
---

The cheap USB NFC reader/writer (app "NFC卡片读写 V2.1", status bar "演示系统"/Demo) is a **write-only NDEF appliance**, not a UID reader. Fully reverse-engineered Aug 2026.

**Hardware/transport:** USB HID, VID_FFFF/PID_0035, "ARM CM0", usage page 0xFFA0, 256-byte **feature reports only** (SetFeature report id 0x01 = command, GetFeature report id 0x03 = reply). Driven by bundled `usb.dll` (generic "generalHID" wrapper: hidWriteData=SetFeature, hidReadData=GetFeature).

**Frame:** `AA <len16 BE> <payload> <XOR of len+payload> BB`, wrapped in report as `[01][00×5][frameLen16 LE][frame…]`. e.g. detect = payload `D9 0E` → `AA 00 02 D9 0E D5 BB`.

**Commands (payload):** `D9 0E`=detect (reply byte0 01=card present, 00=absent), `D9 01 <len> 02 <lang> <text>`=write NDEF text, `D9 00..06/0F`=RF config, `0x88 <p1> <p2>`=LED, **`0x89 <p1> <p2>`=BUZZER (needs BOTH param bytes; `89 01 00` turns it on = a repeating pulsing buzz; for ONE short chirp send `89 01 00` then `89 00 00` ~80ms later. Params don't set freq/duration.)**, 0x80=addr,0x81=baud,0x82/0x83=serial. Replies are always a 2-byte status `01 XX` (83 detect, 8C write, 89 LED/buzz, 85 read-fail); **the reader never returns card data/UID**. `0x1B` (app's read-uid) and `0x25` (pyrfidhid CMD_READ_TAG) both return status only on this 13.56 MHz firmware.

**Same protocol family as [charlysan/pyrfidhid](https://github.com/charlysan/pyrfidhid)** (its 125 kHz EM4100 cousin DOES return UID via 0x25 + GetFeature report id 0x02; the 13.56 MHz NFC variant does not).

**Key gotcha:** the reader only truly executes writes/buzzer inside the vendor app's session; standalone, detect + buzzer + LED work reliably but writes return `83` (not `8C`). The app crashes ("RichEdit line insertion error" + mojibake) because system ANSI codepage is **65001/UTF-8** (the "Beta: Use Unicode UTF-8" option) instead of GBK 936.

**Deliverable:** `Downloads/nfc-scanner/nfc_scanner.py` — standalone Python (ctypes, no deps) tap-detect + reader-beep scanner. For real per-card identity (check-in), need an ACR122U/PN532 instead. Capture shim (`usb.dll` proxy + `usb_real.dll`) left deployed in the app folder unless restored. See [[schedulle-project]].
