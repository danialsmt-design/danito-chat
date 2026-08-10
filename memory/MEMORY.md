# Memory index

- [User: Danial](user-danial.md) — SMT production engineer in Bangi; "Danial" = the person on the "Lourdes Gunadasan" account.

- [Schedulle project](schedulle-project.md) — team check-in tracker: Node server + PyInstaller tray-agent exe with idle tracking.
- [NFC HID reader protocol](nfc-hid-reader-protocol.md) — VID_FFFF/PID_0035 demo NFC writer: full reversed protocol; buzzer/detect work, no UID; standalone scanner at Downloads/nfc-scanner.
- [RFID app (PN532 tool)](rfid-app-pn532-tool.md) — dump/crack/write/clone web app for the PN532 (COM4); test card is gen2 magic; UHF reader integration pending.
- [MCS / MCS Ai](mcs-material-control-system.md) — the new central material control system; why not "PCS", and where the "Ai" belongs.
- [Sony SMT parts verification](sony-smt-parts-verification.md) — parts-exhaust interlock: 6 lines × 4 Sony F130/F209 mounters, 1 PC/line over serial, scanner-based reel check.
- [Parts Control DB write access](parts-control-db-write-access.md) — dbsvc CAN write data remotely (no DDL, so back up to CSV not a table); ACER-PC (dbo) only for grants.
- [Reading true stock](reading-true-stock.md) — StockIns.RemainingQty is zeroed on issue; store-only stock reads zero for parts on the feeders.
- [Remote access from home](remote-access-from-home.md) — Tailscale double-hop: home → line1pvs (100.108.0.118) → LAN → Parts Control PC (192.168.0.134).
- [Parts DB server migration](parts-db-server-migration.md) — moving ReelPart-New off the shop-floor desktop to a real server; DB hardening (backups, audit cols, constraints, LineId) queued for then.
- [Parts Control PC health](parts-control-pc-health.md) — 4GB Acer, C: was 0.1% free, ReelPart-New unbacked-up since 14 May 2026.
- [Canon part invoices](canon-part-invoices.md) — OCR'd Mar–Jul invoices (RM 6.35M); prices are stable; the DB price master was fixed from them.
- [Canon board costing](canon-board-costing.md) — per-model material cost from ProductBOM + the four traps that make a naive SUM wrong.
- [Canon BOM source of truth](canon-bom-source-of-truth.md) — ProductBOM.Quantity lies (L254 said 31, truth 1); read the MSPC PDFs instead, password = model number.
- [PVS model naming](pvs-model-naming.md) — only Line 3 uses a "L3" variant; all other lines share the common un-suffixed model; amendments go into the common product.
- [PVS shift + daily report](pvs-shift-and-daily-report.md) — auto shift-change at 07:35/19:35, per-shift status reset, downtime tracking, /report.html; DB date/shift format quirks (DPC yyyy-MM-dd/"Morning", ProductionLog dd-MM-yyyy).
- [PVS Line 2 deploy](pvs-line2-deploy.md) — reaching LINE2PVS (Tailscale 100.94.102.44, line2pvs\Administrator) + the monitor-only pilot deploy (self-contained, no runtime install).
- [PVS Line 5 deploy](pvs-line5-deploy.md) — LINE5PVS pilot (Tailscale 100.101.8.76, line5pvs\Administrator); A-side; COM cables to be swapped to Line-1 order later.
- [PVS connection guardian](pvs-connection-guardian.md) — netguard: resident self-healing watchdog on each line PC; fixed the Line 5 static-Wi-Fi (no gateway) outage that blocked operator-ID scanning.
- [PVS C1M machine counter](pvs-c1m-machine-counter.md) — reading the Sony completed-PWB counter over serial is rejected (A4E00) by M4; machine-side config, not our code; reader built + safe, waits on machine HMI.
- [PVS count: two counters + cap](pvs-count-two-counters-and-cap.md) — never-reset serial report counter vs operator-reset panel count; trusting the wrong one inflated a lot; adoption now capped at target (Tier 1 live on all lines).
- [PVS substitute parts](pvs-substitute-parts.md) — a slash in a BOM/feeder part number = substitute; PartsMatch is exact-only so it false-rejects valid substitutes; fix pending exact-format confirmation.
- [Second brain vault](second-brain-vault.md) — Obsidian PARA vault at Documents/Dantec/DANOTTO; Claude does the Organize+Distill step.
- [Claude transcript backups](claude-transcript-backups.md) — 2h scheduled export to Documents/Dantec/ClaudeBackups, after a zeroed index file hid a chat.
- [Gmail→WhatsApp digest](gmail-whatsapp-digest.md) — 2-hourly Gmail summary sent to WhatsApp self-chat (60122185237); needs the local WhatsApp bridge running.
- [Bursa trading agent](bursa-trading-agent.md) — 0.469% round-trip cost floor killed every intraday backtest; buy-and-hold beat all variants; Bursa data-licensing traps.

- [PC health: known issues](pc-health-known-issues.md) — Dell Inspiron 7490; Dell ServiceShell RAM leak + a run of 3 BSODs to keep watching.
- [Thunderbird nstmp disk bloat](thunderbird-nstmp-disk-bloat.md) — failed-compaction temp files silently ate 195GB of C:; will recur, check here first when disk is full.
- [Windows startup mechanisms](windows-startup-mechanisms.md) — four places to check, not one; Store-app startup tasks are the one everyone misses.
