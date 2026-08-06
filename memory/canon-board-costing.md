---
name: canon-board-costing
description: "Per-board material cost by model from ReelPart-New BOM, plus the data traps that make a naive SUM wrong."
metadata: 
  node_type: memory
  type: project
  originSessionId: 8eccac87-855c-43db-afb3-57bccac03906
  modified: 2026-07-30T09:22:14.050Z
---

Costing Canon boards from the **ReelPart-New** DB (Parts Control PC). Tables: `ProductBOM` (Quantity/UnitPrice/TotalPrice/Side/Line), `PartPrices`, `Products`, `ProductSellingPrice`, `DeliveryDocuments`. Derived 2026-07-30.

**Material cost per board (RM)** — SMT A+B side plus the line-less "Full" rows (bare board / major assemblies):
L254 26.44 · L261 26.55 · L264 26.41 · L307 24.19 · L309 31.73 · L311 33.90 · L313 24.19.
Selling prices in `ProductSellingPrice`; GP runs 19–39%, blended ~21%. L347 has a selling price but **no BOM rows**; the `PBA*`/`PD-*` products have neither.

**Four traps — a plain `SUM(TotalPrice) GROUP BY ProductID` is wrong:**
1. **BOMs are stored once per line and the copies disagree.** L261 sits under Line 1 and Line 3; L311 under Lines 1, 2, 4. Line 1's copies have qty inflated (×2/×4) and zero UnitPrice on parts that *do* have a `PartPrices` row. Use **Line 3 for L261/L264/L254, Line 2 for L311** (Lines 2 and 4 are identical).
2. **Bare boards are double-listed**: a `YH4-xxxx-000` placeholder at RM 0 on the B side, and the priced `YH4-xxxx-008` in the line-less Full list. Count only the `-008`. **`YH4-3225-008` is misfiled under L307 but belongs to L311** (L311 holds the `-3225-000` placeholder) — uncorrected this shows L307 at −19.9% GP and L311 at 51.6%. The four `-008` rows postdate the `ProductBOM_backup_20260727` table, so it's a recent entry slip. Fix not yet applied.
3. **`VSI-` is a typo for `VS1-`** (letter I vs digit 1) — `VSI-9243-018`, `VSI-9270-010`, `VSI-9540-012` are duplicate rows of parts already in the Full list. Exclude, don't backfill.
4. **`DeliveryDocuments.UnitPrice`/`TotalAmount` are all zero/NULL** — never populated. Value POs at `ProductSellingPrice × Quantity`. `Status='Planned'` = not yet shipped. **`DocType` is NOT a usable filter**: the Canon L-series is tagged `Receive` Mar–May, mixed in June, `PO` by July — same shipments throughout (each row has its own `HC207…` PO number and date). Only `PBA*` receives are genuinely inbound.

**Date traps:**
- **`DeliveryDocuments` dates are wrong before June 2026.** The module went live 20 May 2026 and someone backfilled history in one sitting — `CreatedDate` is 2026-05-20 for *every* March and April row. They finished Mar (39 docs) and Apr (68), entered only 11 of May's, and stopped. So May reads RM 134k against RM 2.78M actually invoiced, and April is inflated to RM 2.54M against RM 1.02M invoiced. PO numbers don't match their dates (`HC20757308000` dated 6 May sits above POs dated 20 May). Aggregate Mar–Jul still reconciles to 2.7% — nothing is missing, it's misdated. **Use the invoice register for anything before June; June onward the DB is trustworthy.**
- **`DailyProductionCount` stores TWO date formats**: `dd-MM-yyyy` up to mid-May 2026, `yyyy-MM-dd` after. Filtering on one silently drops the other — this made me report that production tracking starts in May when it actually starts **late March**. Normalise with `CASE WHEN Date LIKE '[0-9][0-9][0-9][0-9]-%'`.

**⚠ Danial's standing instruction (2026-07-30): use ONLY data in the DB for these figures — no spreadsheets.** Say "no data" rather than substituting another source.

**Pre-March 2026 the DB is empty.** Dec'25 and Feb'26 have zero `DeliveryDocuments` rows; Nov'25 and Jan'26 have three `PBA0484A:A` receives (`PO0286xx`, not the Canon `HC207…` series) with no BOM and no selling price. `DailyProductionCount` has zero rows before March 2026; `ProductionLog` covers only July 2026. So **Dec/Jan/Feb sales and cost are not answerable from the DB** — that ~RM 166k of business exists only on invoices.

Cross-check that exists but is off-limits by default: the `INV GMS` sheet of the CANON CONTRA workbook. Invoice amounts decompose uniquely into model × qty because the selling prices are distinctive (`9,229.05 = 300 × L313`). Only reach for it if Danial asks.

**Inventory / stock on hand — the ONLY correct rule: `SUM(StockIns.RemainingQty)`.** Depleted reels carry `RemainingQty = 0`, so no dedup or date filtering is needed. Verified against the floor: `YH4-3081-000` = 2,663 pcs (4 reels, all stocked in 28-07-2026), which Danial confirmed as the physical balance. Canon total 2026-07-30: **113 parts, 240 reels, 1,032,305 pcs, RM 250,652.65** (88,687 pcs unpriced). Note `Status` is not the filter — plenty of `Status=1` rows have `RemainingQty=0`.

Two methods that look right and are **wrong**:
- *Sum `StockOuts.Quantity`* — it's a **balance snapshot, not an issue quantity**; successive rows for one `PartUID` restate the reel's remaining qty (`0000-A827%`: 8000 → 6495 → 5204 → 4804). Taking the latest row by `ID` still fails because a reel is only rescanned on model change/verification, so stale readings inflate on-hand to ~RM 5.2M.
- *Received − consumed* (`SUM(StockIns.Qty)` minus production × BOM) — gives RM 1.58M, ~6× too high, because **`StockIns` is not receipt-only**: the same `PartUID` is re-stocked-in each time a reel returns from the line, so `SUM(Qty)` counts the same physical material repeatedly (`1000-C163%` has 4 stock-in rows, `081000-G015%` has 3).

Stock movements only start Feb 2026 (no opening balance); `WasteLog` is empty so no scrap is recorded.

`CanonSentMonthly` = consigned parts Canon sent to GMS, loaded from `CANON INVENTORY JUNE.xlsx` (161 parts, June 2026 only) — this is the "canon inventory" file, not a BOM.

**Also unpriced:** `YH4-3057-000` on L254 (no `-008` counterpart, absent from `PartPrices`), so L254's cost is understated. 24 BOM part numbers have no `PartPrices` row; 11 have a BOM UnitPrice disagreeing with `PartPrices`.

**Validation that works:** July'26 delivered = RM 2,176,078 computed vs RM 2,172,771 in the `CANON CONTRA` workbook's PENDING SUMMARY (0.15% apart). Those `CANON CONTRA *.xlsx` files in Downloads are contra/invoice reconciliation, **not** inventory or BOM. `CCT PO *.xlsx` is a different customer in USD — Danial said to ignore CCT.

All figures are material only — no labour, machine time, scrap or overhead. Read the DB read-only over the Tailscale WinRM path in [[remote-access-from-home]]; see [[parts-control-db-write-access]] before any write.
