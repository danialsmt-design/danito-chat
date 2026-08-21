---
name: mcs-material-control-system
description: MCS / MCS Ai — the new central material-control system replacing the vendor parts-control app; naming decision and rationale.
metadata: 
  node_type: memory
  type: project
  originSessionId: 45a3a5d4-9c33-4440-ac3d-328171bfb409
  modified: 2026-07-31T15:49:04.201Z
---

**MCS = Material Control System** — the new central system for the parts/material side: reel
inventory keyed by reel UID, goods-in, stock-out to lines, BOM, badge master, production count.
The central counterpart to [[sony-smt-parts-verification]] (PVS = the line-side verification /
interlock app). Danial named it on 2026-07-31.

**Naming history — do not re-litigate:**
- Danial first proposed **PCS (Parts Control System)**. I argued against it: in SMT, "PCS" already
  means *pieces* (every reel label reads `3000 PCS`) and it collides with Process Control System.
  Naming the quantity system "PCS" gives you "PCS shows 4000 PCS".
- Settled on **MCS** because it is already the floor vocabulary — 192.168.0.134 has been called the
  "material control PC" throughout the design — so it costs nothing to adopt.
- Danial then asked for **"MCSAi"**, explicitly as a tribute to Claude / "the future of cyber
  machines". I flagged one risk and proposed a split rather than refusing the name.

**✅ CONFIRMED by Danial 2026-07-31: use the split below** ("the naming i will follow your
sugestion"), and **MCS is a fresh build**, not a layer on the vendor app — data to be migrated off
the Acer.

**The split (confirmed):**
- **MCS** = the system. Deterministic, auditable, boring on purpose.
- **MCS Ai** = the intelligence layer only (the Phase 9 work: learned per-feeder consumption rates,
  learned board rates per model, anomaly detection, Mode D drift patterns).

**Why:** Danial's own hard rule is that *the interlock stays deterministic and never learns —
right part or stop*; learning belongs in prediction/warning only, never the safety decision. A
whole-system "Ai" name invites a customer/Canon auditor to ask "so the AI decides if the part is
correct?" and oversells it. The split keeps the safety path honestly non-AI while putting the name
where real intelligence exists. **If Danial reaffirms MCSAi for the whole system, use it
throughout without further comment** — his factory, his call.

**💰 COMMERCIAL MODEL = BUY-SELL / TURNKEY (Danial, 2026-07-31 — FINAL, supersedes two wrong readings
of mine).** **Canon invoices GMS for every component SENT (60-day terms); GMS invoices Canon for the
finished boards (60-day terms).** So it is NOT consignment and NOT pay-on-consumption — **GMS OWNS the
inventory from the moment Canon ships it**: GMS's stock, GMS's balance sheet, GMS's cash. (I said
first "loss is Canon's material, not yours", then "pay-on-consumption / report is the billing
trigger". Both WRONG. Do not repeat either.)
Consequences:
1. **Component loss is PURE MARGIN LOSS** — material GMS already paid for that never became a
   sellable board. No allowance to sit inside, nobody to bill it to.
2. **WORKING CAPITAL:** 60 days both ways, so every day material sits in the store is a day of GMS
   cash tied up. ⇒ **over-requesting is borrowing, not just clutter** (I initially called
   over-requesting merely inconvenient — wrong).
3. **GMS's BALANCE-SHEET inventory figure traces back to those twice-monthly counts** — MCS improves
   the accuracy of a number in the financial statements. Different, more senior audience.
⇒ **The ledger must carry VALUE, not just quantity.** Loss reported in currency and per feeder;
actual-vs-standard material cost per model (standard from `ProductBOM` — see [[canon-board-costing]] —
the gap IS the loss); continuous inventory valuation; and **reel AGEING** from `StockIns.Date` (dead
stock = GMS cash frozen; a report they almost certainly do not have today).

**📉 TWO INDEPENDENT BUSINESS CASES (use both with GMS):**
1. **Downtime** — the physical count is **2 h × twice a month, ALL LINES STOPPED, whole floor
   counting** = **48 h/yr of full-factory stoppage = 288 line-hours**. At ~53 machine cycles/h × 4
   boards/panel (L307) ≈ **210 boards/line-hour** ⇒ order of **60,000 boards/yr**. MCS can honestly
   claim 24 counts/yr → 1–2, and **no line stoppage** (store-only, cycle-counted while running using
   the `PartRanks` A/B/C that already exist). Manual counting never reaches zero — it becomes the
   monthly reconciliation ANCHOR that proves the ledger.
2. **The cash leak** — unmeasured component loss on billed-back material. Probably the bigger number.

**🏗️ ARCHITECTURE BOUNDARY (Danial, 2026-07-31 — explicit):** **PVS owns everything that touches a
machine** (serial, Sony protocol, R0/R2/C1M/**C1Z**, verification/interlock; one instance per line).
**MCS never talks to a machine** — it consumes what PVS publishes, keeps the ledger, produces the
views. **PVS must persist an append-only consumption log and MCS pulls INCREMENTALLY BY CURSOR**, so a
line-PC outage backfills instead of losing production. See [[pvs-c1m-machine-counter]].

**📋 MCS v1 = READ-ONLY ANALYTICS (Danial's choice).** Writes nothing to `ReelPart-New`. Ledger types:
Received (`StockIns`) · Issued (`StockOuts`) · **Consumed** (measured Line 1 / backflushed lines 2–6)
· Returned · Adjusted (`ReelAdjustments`) · Scrapped · CountVerified. **Every row carries PROVENANCE**
(machine-measured / inferred / backflushed / human-keyed) — with pay-on-consumption billing that
provenance field IS the audit trail when Canon queries a number. **Hosted on `LINE1PVS`** (7.4 GB RAM,
182 GB free, always on, DB over Tailscale, proven self-contained .NET 8 deploy) — **explicitly
temporary** until the server in [[parts-db-server-migration]] exists.
**🎯 DEADLINE + PROOF: next physical count is MID-AUGUST 2026 (~14 Aug).** MCS v1's first deliverable
is NOT a dashboard — it is a **variance report**: MCS's computed balance vs what the count finds. The
honest scope for that date is Line 1 only (the measured line). NOT achievable by then: C1Z on all 24
machines, or measured consumption on lines 2–6 (needs PVS rolled out).

**Open, and more consequential than the name:** is MCS a *replacement* for the vendor ASP.NET app
(`ProductionAPI` + `ReelPart-New`) on the Acer, or a *new layer beside it*? This drives the DB work
already queued in [[parts-db-server-migration]]. Given the Acer's state ([[parts-control-pc-health]]
— 4GB, C: at 0.1% free, 10 weeks with no backup), "replace" looks like the honest answer.
Related: [[parts-control-db-write-access]].

**🚀 SCOPE EXPANDED 2026-08-13 — MCS = the FULL operational system, not just read-only analytics
(supersedes the "v1 = read-only" line above).** Danial wants MCS to own the whole reel lifecycle:
**new-parts registration → mint reel UID → print label → stock-in → stock-out**, PLUS the analytics.
So MCS **replaces the WinForms `PartsControlSystem_App`** and becomes a *writer* to `ReelPart-New`
(fold in the DB hardening from [[parts-db-server-migration]] — audit cols, unique constraints,
part-number validation — at that point). Maps to tables: registration→`Parts`(+`PartPrices`),
stock-in→`StockIns`, stock-out→`StockOuts`, UID = per-reel `PartUID`.
**Architecture (confirmed):** self-contained **.NET 8 web app hosted ON THE NAS** (same proven
pattern as PVS), reading/writing `ReelPart-New` on the NAS. Operators use a browser on the
**"MCS terminal" = the existing Parts PC `.134` (DESKTOP-TECHNIC)** — which transitions from DB-host
to a thin client (browser + printer + scanner) once the DB is on the NAS.
**Label printing:** Zebra **ZD230 (203 dpi, ZPL), USB** on `.134` → **Zebra BrowserPrint** (small free
local service) so the web page sends ZPL over `localhost` to the USB printer. Barcode **scanner = HID
keyboard**, no integration. TODO before building: exact **label stock size** (read off `.134`'s
printer config / current template), and design the **reel-UID scheme** (today's UIDs look like
`355123-H123%` = part-derived; needs a robust unique scheme). Data quirks to respect (from NAS-copy
profiling 2026-08-13): `StockIns.RemainingQty` is zeroed on issue (don't use for balance); dates are
**nvarchar `dd-MM-yyyy`**; value via `PartPrices.UnitPrice` by `PartNumber`; ~8084 reels in / 7575 out.
Store-count **variance report = received − issued + returns ± adjustments** (store-only count → no
machine-consumption needed for that report).
**🎨 UI DIRECTION APPROVED 2026-08-13:** clean/industrial, self-contained HTML/CSS/JS (no framework —
right weight for the NAS + factory box), cool-neutral palette + single teal accent, mono/tabular figures
for UIDs·qty·price, uppercase labels; live ZD230 label preview (50×25 mm) with a rendered barcode. The
**stock-in mockup is the reference design** (artifact `claude.ai/code/artifact/c0b74804-e2dc-4377-9042-6b13440124c3`;
source in this session's scratchpad `mcs-stock-in.html`). Danial confirmed the style fits the shop floor.
Proposed reel-UID scheme in the mockup: **`RP-YYMMDD-####`** (prefix + date + daily running seq) — cleaner
than today's part-derived `355123-H123%`; still to be ratified. Stock-in fields shown: part (scan/autocomplete
from validated master → auto maker/price/rank/in-stock), order#, qty, auto UID, auto received-time; primary
action "Mint UID · Print label · Save". Charts/Reports page will use the `dataviz` skill later.
**📐 MCS DATA/BUSINESS RULES (ratified by Danial 2026-08-13):**
- **MCS extends the EXISTING `ReelPart-New`** (26 base tables, on the NAS) — NOT a new DB. "Build the MCS DB" = extend + harden it.
- **Returns → reactivate the ORIGINAL reel** (same `PartUID`, restore remaining qty to store, mark not-consumed). ⇒ **UID is minted ONCE at first stock-in and never regenerated** — one stable UID per physical reel for life.
- **"True stock" = STORE + ON-LINE** = reels in store (received/returned, not issued) + reels on feeders (`InLineInv`) − waste/adjustments. Reconciles vs the current whole-floor physical count.
- **UID scheme RATIFIED = `RP-YYMMDD-####`** (Danial 2026-08-13; prefix + mint-date + zero-padded daily running seq). Minted once at first stock-in. Legacy reels keep their `Counters`-derived UIDs (`355123-H123%`); both coexist (PartUID is a string) — no renumbering.
- **Key schema facts:** `InLineInv`(Line,PartUID,Qty)=on-feeder reels; `Parts`(PartNumber,Maker,Quantity,Available bit,IDCounter); `PartRanks`(PartNumber PK,Rank char); `Users`(UserID,AccessLevel,UserName,Password **plaintext nchar(10)**,UserUID badge); `PartPrices`(PartNumber,UnitPrice).
- **Hardening gaps found (fix as MCS writes):** NO foreign keys anywhere; **`StockIns` and `Parts` have NO primary key**; plaintext passwords. These let the bad data in (typos, dup BOM rows).

**✅ STOCK-IN PAGE BUILT + LIVE ON THE NAS (2026-08-14).** First real MCS page working end-to-end. Stack: **.NET 8 minimal-API** (thin, `Microsoft.Data.SqlClient`, raw ADO.NET — no EF) in a **Docker container `mcs-web` on the NAS** (multi-stage build; source at laptop `Documents/Dantec/MCS/app/` → NAS `/volume1/docker/mcs-web/`, `docker compose up -d --build`). Serves at **`http://192.168.0.169:8090`** (LAN) / **`http://100.125.22.119:8090`** (Tailscale). Connects to the **STAGING copy** (`Server=192.168.0.169,1433` as `partcontrol_user`/GMS2026, via `MCS_DB` env in compose). Endpoints: `/api/health`, `/api/parts` (320 parts + price + rank), `/api/stockin` (validates part exists → mints `RP-YYMMDD-####` daily-incrementing in a txn → INSERT StockIns Status=1 RemainingQty=Qty), `/api/recent`. **Verified:** minted RP-260814-0001/0002, UID increments, unknown part rejected, recent list works. Label: builds ZD230 **ZPL** + best-effort **BrowserPrint** POST to `localhost:9100` (degrades until BrowserPrint installed on .134). **Deploy cmd (Synology): NO `cpus:` in compose (kernel rejects NanoCPUs); mem_limit ok.** Two test StockIns rows on staging (throwaway).
**✅ ALL CORE PAGES BUILT + LIVE (2026-08-17):** stock-in, **stock-out** (scan UID→issue to line→INSERT StockOuts), **register** (new part→Parts/PartPrices/PartRanks, validated, no-dup), **returns** (reactivate reel via ReelAdjustments — ⚠️ STAGING-ONLY banner, replaces Ashish's clear page per COORDINATION.md), **reports** (Canon order book ordered/delivered/outstanding + delivered RM from the live mirror; Canon parts-invoiced value from `MCS` db). Shared `mcs.css` + linked nav. New endpoints in `Program.cs`: /api/lines /api/models /api/reel/{uid} /api/stockout /api/register /api/return /api/reports/orderbook /api/reports/canoninvoices. **Reports/canoninvoices reads the separate `MCS` db → had to `CREATE USER partcontrol_user FOR LOGIN` + db_datareader/writer in MCS** (login only mapped in ReelPart-New before). Full register→stockin→stockout loop verified end-to-end. **✅ PO MONITOR ABSORBED into MCS (2026-08-17):** Danial had MCS take over the existing **"PO Monitor"** tool (was `C:\Users\ACER-PC\Desktop\PoUpdate.html` on .134 → a big .NET `ProductionController.cs` API on `192.168.0.134:54063`, part of ProductionAPI which ALSO does production sessions/dashboard/Arduino — MCS absorbs ONLY the PO slice, ignore Arduino). New MCS **PO page** (`/po.html`): Monitor (KPI cards Total/Overdue/Partial/Delivered/OnTrack + filter + per-PO **Deliver**) and **Add PO**. Endpoints: `/api/po/monitor` (DeliveryDocuments enriched with ProducedQty from DailyProductionCount where LotNo=PONumber+Model), `/api/po/add` (INSERT DeliveryDocuments, skip-if-exists per PO+model), `/api/po/deliver` (accumulate DeliveredQty → Status Delivered/Partial). Writes `DeliveryDocuments` → ⚠️ **the live PO Monitor on .134 still owns this until cutover; MCS's writes are staging/ephemeral (mirror overwritten hourly).** **✅ production-slip + tray-label DONE (2026-08-17):** `/slip.html` = Production Identification Slip, one label per magazine (`computeSlipMagazines`: magCount=ceil(poQty/magQty)), 6/A4, QR (qrcode-generator CDN) encodes `PO: <po> | QTY: <poQty>`; fields Customer/Line/PO+Side/PO-qty/Model/Prod-date/mag-qty/AOI-VI/QC/Remarks. `/tray.html` = GMS BANGI box labels (`computeTrays`: boxCount=ceil(total/perBox)), Customer/Model+`(code+"000")`/PO/QTY/Box, L307 code highlighted yellow, last box "(DOCUMENT INSIDE)" optional; product code via `/api/product-code` (Products→ProductBOM, e.g. L307→YG2-4968-008). Both A4 print via `window.print()` (NO BrowserPrint needed — that's only for ZD230 reel labels on stock-in). Linked from po.html subtabs. **PO Monitor fully absorbed.**

**Shop-floor access (2026-08-18):** MCS web app serves on NAS `mcs-web` container, port 8090. Reachable from any shop-floor PC three ways: LAN `http://192.168.0.169:8090/...`, Tailscale IP `http://100.125.22.119:8090/...`, or **friendly name (MagicDNS is ON, suffix tail14c782.ts.net)** `http://gmssmt1:8090/...`. Each page is its own address: `/po.html`, `/slip.html`, `/tray.html` (+ `/`, `/stock-out.html`, `/register.html`, `/returns.html`, `/reports.html`). Added **`/go.html` = shop-floor launcher** (big-button home → PO/Slip/Tray + material pages; relative links so it works via any of the three hostnames). Bookmark `/go.html` as the floor home.

**Remote deploy channel to NAS (no plink; works over Tailscale):** SMB share `\\100.125.22.119\docker` (cred `nas.cred` = GLOBALSMT, in administrators). App source at `docker/mcs-web/` (`build: .`, Dockerfile bakes wwwroot — so copy file to `mcs-web/wwwroot/` for permanence). Run commands via Windows `ssh.exe` with **forced askpass** (`$env:SSH_ASKPASS`=cmd that types the pw, `SSH_ASKPASS_REQUIRE=force`); docker needs sudo + full path `/usr/local/bin/docker`, feed sudo pw via `sudo -S -p '' < pwfile`. Live-deploy a static file without rebuild: `docker cp mcs-web/wwwroot/X mcs-web:/app/wwwroot/X` (survives restart/reboot; next `--build` bakes it from the staged source). See [[nas-reelpart-db-host]].

**DONE (2026-08-18) — Lot Completability feature** (`/completability.html` + `GET /api/completability?month=YYYY-MM`, default current month). Runs MRP: outstanding Canon boards (order−delivered, that month) → explode ProductBOM → vs stock (StockIns.RemainingQty + InLineInv) → shortfall, cross-referenced to `[MCS].dbo.CanonPartInvoices` (that month) for cover=covered/partial/none + Description/Origin/UnitPriceRM. Returns summary{outstandingBoards,partsNeeded,partsShort,shortfallValueRM,onInvoice,notOnInvoice} + byModel + shortParts[]. Page: verdict banner (good/warn/crit), stats, "gaps only" filter, 20-min auto-refresh. **Forward-looking** (2026-08-18 update): param is `from` (YYYY-MM, default current month); window = due `>= from` open-ended (this month + all future, NOT a single month), Canon invoices `AccountingMonth >= from`. Response adds `byMonth[]` breakdown. Page picker is a "From month" input with min=current month (forward only). Aug-forward baseline: 25,200 boards (Aug 22,200 + Sep 3,000). Added Completability tab to all pages' nav + launcher tile. Aug'26 baseline: 22,200 boards, 126 parts, ~92 short, ~RM526k, 82 on-invoice/10 true gaps. Endpoint uses Conn() (ReelPart-New) with 3-part MCS refs (partcontrol_user has MCS access). Deploy = image rebuild (`docker compose up -d --build`) since Program.cs compiled in.

**ROLLED BACK — the instrument-panel redesign** (user: "this looks bad"). mcs.css restored from `mcs.css.pre-design.bak`; po.html instrument bits (DIN .st, rail, ticks) stripped, kept the approved dataviz meters + "delivered vs outstanding" overview. Now live. The frontend-design plugin exists but user rejected the distinctive direction for this shop-floor tool — keep the plain original teal/rounded system.

**(superseded) app-wide "instrument-panel" visual identity** (via `frontend-design` plugin skill + `dataviz`). Rewrote shared `mcs.css` keeping all class/variable names so all 8 pages reskinned with no markup breakage: DIN/**Bahnschrift** display type (built into Win10/11 → local on every line PC, no web-font risk), sharp corners (`--r:3px`), cooler engineered palette (teal `--accent` kept for continuity; `--good` phosphor-green, `--warn` amber), instrument-style stat readouts, ink-filled active tabs, DIN uppercase labels/badges. Signature elements = **segmented (ticked) fulfillment gauges** + **status rail** (coloured left border on each PO lot row, `.r-<status>`). PO page also has produced/delivered meters + "Delivered vs Outstanding by board" overview chart. Backup of old stylesheet at `wwwroot/mcs.css.pre-design.bak` (and on NAS) for instant rollback. All live + verified HTTP 200. Design was approved by user via a standalone preview first (`scratchpad/mcs-po-design.html`).

**Completability demand = BOARDS TO BUILD (2026-08-18):** changed `/api/completability` (+ `/months` + byModel) demand from `order−delivered` to **`order − max(delivered, completed-both-sides)`** per lot — excludes SMT-completed boards (produced but not delivered) since their material is already consumed. `completed` per lot = `CASE WHEN Full>0 THEN Full ELSE min(sumA,sumB)` from `DailyProductionCount` (Side A/B/Full). `max(delivered, completed)` because production count is INCOMPLETE for some lots (e.g. L254 produced 380 but delivered 900) — delivered is the safety floor so we never over-state "done". Page relabeled "Outstanding boards"→"Boards to build"; "Check now" button→"🔄 Refresh" (reloads months+check). Aug effect: 18,600→**7,256 boards**, shortfall RM 259,766→**90,189** (combined with the BOM fix, down from the original RM 515,852). Caveat: leans on PVS-owned production count. Manual DB refresh (`refresh.sh`) can be run on demand for latest .134 data.

**BOM double-count fix in completability (2026-08-18):** `ProductBOM` has duplicate/phantom rows that inflated the completability "needed/short". Two patterns: (1) exact-duplicate placement rows (60 groups, 67 extra rows overall); (2) **phantom 'Full' board-level rows** (Side='Full', Machine=0, no SupplyPosition) that duplicate a real SMT feeder placement of the same part (e.g. YH4-3216-008 on L309 had a real B-side/machine3/feeder636 row + a 'Full' row → counted 2× instead of 1). ~49 of 126 Aug-demand parts affected. **Fixed in `/api/completability`** (Program.cs `baseCte`): added CTEs `dr` (DISTINCT ProductID,PartNumber,Side,Machine,SupplyPosition,Quantity — collapses exact dupes), `hasreal`, `kept` (drop Side='Full' rows when a non-Full row exists for that part, else keep Full), `bomcorr` (SUM per part); `partneed` now joins `bomcorr` not raw ProductBOM. Read-only correction — does NOT modify ProductBOM (it feeds production). Result Aug: partsShort 84→72, shortfall RM 515,852→259,766 (~45% was phantom), YH4-3216-008 14,400→7,200 (validated). Legit high-count parts (e.g. WA2-2419) unchanged. Deployed via image rebuild.

**Completability ON-HAND redefined + SIDE-AWARE demand (2026-08-20, Danial-directed):** Two structural fixes to `/api/completability` `baseCte` (Program.cs), both deployed + verified live.
(1) **On-hand = STORE + STOCK-OUTS, dropped InLineInv.** Danial: "stocks out = spare at line + loaded on machine". `stock` CTE now = `StockIns.RemainingQty` (in store) + latest-per-UID `StockOuts.Quantity>0` (issued/at-line), NOT `InLineInv`. DB check: the three buckets are DISTINCT reel sets (only 11-13 UID overlap) — store 1.32M, stock-out 1.23M, InLineInv 0.875M. Aug effect: short 42→39, RM 90,189→22,098.
(2) **SIDE-AWARE demand** — a half-built board (one side already run) only re-demands the parts for the side it still needs. `demandSide` CTE computes per lot: `a_work=Ordered−max(Delivered, A_prod+Full_prod)`, `b_work=Ordered−max(Delivered, B_prod+Full_prod)`, `f_work=Ordered−max(Delivered, completed-both-sides)`. `bomside` keeps the placement Side ('A Side'/'B Side'/else=Full → aqty/bqty/fqty). `partneed = Σ a_work×aqty + b_work×bqty + f_work×fqty`. Hand-verified: VE4-2850-104 need 165,280→**88,060** (L309 B-side 23/board is ~99% done → only 700 B-boards left vs 2,392 A-boards). Aug effect (with fresh data): boards 7,256→5,054, short 39→25, RM 22,098→7,570, 6 true gaps.
**Key data-quality findings (Danial's deep-dive on VE4-2850-104 + WA6-6717-000):** (a) Canon invoice ≠ future inbound — invoiced material already stocked-in is already in on-hand; the old green "covered 5×" verdict was misleading — REPLACED 2026-08-20 with Canon-supply-vs-required: badge `cover` now ok(inv≥need)/under(0<inv<need)/none(inv=0), column "Canon supply (vs need)", verdict counts the `none` parts as the true procurement gaps. Green now means "Canon supplies this part" NOT "shortfall covered". Aug: 19 ok / 0 under / 6 none (YH4-3068-000, VS1-8550-005, VS1-9243-008, VS1-8550-007, VR8-1300-331, VE4-5180-104). (b) VE4-2850-104 Aug: Canon supplied 435,000 (invoice) but FULL requirement 722,400 (19,800 L309×28 + 4,200 L311×40) → Canon under-supplied ~287k; July carry-over was ~110,082, all consumed. (c) **StockOut/ConsumedReels tracking is unreliable for some parts** — WA6-6717-000: Canon supplied 68,336 lifetime, only 24,776 used in boards, on-hand shows 180 (+4,913 InLineInv) → ~38k UNTRACKED (drained StockOuts w/ no detail, 0 ConsumedReels rows). So a "short" can be a tracking artifact, not a real Canon shortage. Supply-vs-required across all 154 Aug parts written to `MCS/canon-supply-vs-req-aug.csv` (44 Canon-none, 98 under-supplied, 12 covered; 76 genuine even after carryover). **DailyProductionCount.Date format is MIXED** (old rows dd-mm-yyyy, Aug rows ISO yyyy-mm-dd) — filter Aug with `Date LIKE '2026-08%'`.

**DB cutover readiness (2026-08-18):** NAS SQL = **SQL Server 2022 Express** (MSSQL_PID=Express) — free & production-licensed, no licence blocker. `ReelPart-New` = **80 MB** (trivial vs Express 10 GB cap). NAS is still a **staging mirror**: cron `0 7-19 * * * root /volume1/docker/reelpart-sql/scripts/refresh.sh` overwrites it hourly from .134; `0 2 * * *` backup.sh nightly. **Cutover switch = disable that refresh line.** STAGED (not run) at `/volume1/docker/reelpart-sql/scripts/cutover.sh` (backs up crontab → final refresh sync → sed-comments the refresh cron `#CUTOVER-DISABLED` → restarts crond) + `cutover-rollback.sh` (uncomments, re-enables mirror). Remaining cutover prereqs (NOT MCS-owned): PVS repoints all writers .134→NAS in one window (line1-6 PCs, PC3 Parts Control) [schedulle session]; MCS Returns page must replace Ashish's manual clear page; flip the "Staging" banners off the app pages post-cut; Danial calls the window. See [[mcs-cutover-readiness]].

**Label printer proven (2026-08-17):** Zebra ZD230 (203dpi ZPL) on `.134` via USB001. Prints via Windows spooler RAW (winspool WritePrinter, datatype RAW) — no BrowserPrint needed for raw ZPL. `.134` (desktop-technical) has WinRM/DCOM/schtasks-RPC all blocked; only reachable channel is **SMB(445)→Service Control Manager** (create one-shot service running `cmd /c powershell -File ...` synchronously). `.134` is LAN-only + sleeps after shift (offline on tailnet).
**Still TODO:** Canon-invoice CAPTURE page (upload PDF→vision extract→save; table+Aug data already exist), and **BrowserPrint not yet running on .134** (label print). Note: entry writes go to the hourly-overwritten `ReelPart-New` mirror → test data is ephemeral until cutover. See [[mcs-cutover-readiness]], [[nas-reelpart-db-host]].
