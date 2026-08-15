---
name: mcs-canon-invoice-capture
description: MCS feature to capture Canon part-invoice PDFs into the DB (CanonPartInvoices table). Scanned PDFs → pypdfium2 render + vision extract. Started 2026-08-15.
metadata:
  node_type: memory
  type: project
---

MCS feature (Danial 2026-08-15): **capture Canon PART invoices from PDF into the DB.** Part of [[mcs-material-control-system]] master-data entry. Canon (supplier) invoices GMS for components delivered (buy-sell inbound side); GMS needs a systematic record by month.

**The data gap it fixes:** DB `CanonSentMonthly` has **June 2026 only** (one-off load from `CANON INVENTORY JUNE.xlsx`). The OCR'd `canon-part-invoices.csv` covers Mar–Jul but **July stops at 17 Jul** (4 DNs, RM 248,961.77). **August not captured** (only the DN xlsx `SC-DN-260806-002`). So no complete monthly Canon-parts value until captured.

**✅ Table created on STAGING DB (2026-08-15): `dbo.CanonPartInvoices`** — Id PK identity; **`AccountingMonth char(7)` = USER-KEYED** (Danial's rule: NOT auto-derived from a 26th cutoff — the operator keys which month each invoice belongs to); DNNumber, IssuanceDate, InvoiceDate, PartNumber, Description, Origin, QtyPcs, UnitPriceRM(18,4), AmountRM(18,2), WeightKg, SourcePdf, CapturedAt/By, Verified bit; **UNIQUE(DNNumber, PartNumber)** anti-dup. (DDL on staging only; real DB via Danial at cutover per COORDINATION.md.)

**Capture pipeline PROVEN (2026-08-15):** the Canon invoice PDFs are **SCANNED IMAGES** (0 text chars, 1 image/page — pdfplumber/pdftotext get nothing) → must render + OCR/vision, not text-parse. Working method: `pip install pypdfium2` (pure-python, NO poppler/download needed) → `PdfDocument(f)[page].render(scale=2).to_pil().save(png)` → **read the PNG with vision** → extract line items. Verified on `July 26.pdf` p1 → `SC/DN/260703-008`, VS1-9174-033 FPC/FFC CONNECTOR, JP, 20,000 pcs, RM 0.3950, **RM 7,900.00** — exact match to the CSV.
- Invoice layout: header has Invoice No (=DN `SC/DN/YYMMDD-seq`), Issuance Date, Invoice Date, consignee=GMS; body table cols **NO · COMMODITY(partnum + desc) · ORIGIN · WEIGHT · QTY(PCS) · UNIT PRICE(RM) · AMT(RM)** · TOTAL. One DN per invoice; a multi-line DN spans several PDF pages.
- PDFs live at `C:\Users\Lourdes Gunadasan\Downloads\canon invoice\Part Invoice\{March..July} 26.pdf` (Mar-Jul only; scanned; ~12-16 pages each).

**TODO:** (1) build the MCS **capture page** (upload PDF → render → vision-extract → operator keys AccountingMonth + reviews grid → save to CanonPartInvoices); production vision extraction needs `ANTHROPIC_API_KEY` on the NAS (see [[mcs-ai-agent-spec]]) or local tesseract. (2) Optionally bulk-load the existing `canon-part-invoices.csv` (Mar–Jul, 1054 rows) into the table as the historical baseline → then "Canon parts value by month" is a DB query. (3) August still needs its invoice PDF/DN captured.
