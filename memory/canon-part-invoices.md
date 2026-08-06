---
name: canon-part-invoices
description: "Canon parts invoices Mar-Jul 2026 extracted by OCR — prices, the DB price fix applied, and what reconciling them revealed."
metadata: 
  node_type: memory
  type: project
  originSessionId: 45a3a5d4-9c33-4440-ac3d-328171bfb409
  modified: 2026-08-05T03:33:00.961Z
---

**Source:** `Downloads\canon invoice\Part Invoice\{March,April,May,June,July} 26.pdf` — Canon Opto
(Malaysia) → Global Manufacturing Solution Sdn Bhd. **Scanned images, NO text layer.** Extraction
recipe that works: `pip install pypdfium2 rapidocr-onnxruntime` (both pure-pip, no system binaries;
poppler/pdftoppm is NOT installed so the Read tool cannot render PDFs), render at scale 2.0, OCR,
group boxes into rows by y. Scripts in the session scratchpad (`ocr_invoices.py`, `parse2.py`).
**Validate every line with `qty x unit = amount`** — that identity caught both OCR damage and my own
parser bugs, and repaired 38 rows (e.g. `8.000`→8,000, `t2.8527`→12.8527, `92.00|`, `319]`).
⚠️ **Repair using the part's OWN established price, not back-computation** — my first attempt
back-computed qty from a misread unit price and invented a qty of 67,550 where the truth was 14,993.

**Result: 1,054 line items · 44 invoices · 164 parts · RM 6,354,521.61 (Mar–Jul 2026), 100% verified.**
Monthly: Mar 778,669.60 · Apr 2,080,034.97 · May 1,182,236.73 · Jun 2,064,618.54 · Jul 248,961.77.
Layout: one invoice per page, `NO | COMMODITY | (ORIGIN) | WEIGHT | QTY (PCS) | UNIT PRICE (RM) |
AMT (RM)`; invoice no `SC/DN/yymmdd-nnn`. Output CSVs live in `Documents\Dantec\MCS\` and were copied
to `Downloads\`: **`canon-part-invoices.csv`**, **`canon-vs-db-prices.csv`**.

**💡 PRICES ARE STABLE: 163 of 164 parts held ONE price across all five months.** The only genuine
variation is `YH4-3081-000` 13.4393 → **13.4677** on a single April invoice (SC/DN/260401-002,
+0.21%); both amounts verify arithmetically. Everything else that looked like a price change was OCR.

**✅ DB PRICE FIX APPLIED 2026-08-01** (approved by Danial, run as `dbsvc` over Tailscale, one
transaction with row-count assertions; backups `C:\PVS_Backups\PartPrices_backup_20260801-184024.csv`
and `ProductBOM_backup_20260801-184024.csv`):
- **`PartPrices` 156 → 185 rows** — 29 INSERTs of Canon-invoiced parts that had no master row
  (they were **58% of spend by value**: `YH4-3212-008` RM 1.40M, `YH4-3216-008` RM 1.15M,
  `YH4-3081-000`, `YH4-3225-008`, the `YH1-*`/`YA2-*`/`WV5-*` families). Plus 2 UPDATEs where the
  master disagreed with Canon: `VS1-8550-005` .0802→**.0764**, `VS1-9243-012` .2616→**.2649**.
- **`ProductBOM` 16 rows repriced** — the 11 zero-priced rows in L261's **Line-1** copy (all had a
  master price), `YH4-3068-000` on both line copies (12.8527→**13.4393**, a copy error confirmed
  against Canon), and the two VS1 corrections.
- **The price master now covers 100% of Canon-invoiced value (was 42%).** `YH4-3057-000`, previously
  unpriced anywhere, is **13.4393** from Canon.
- **147 of 164 parts matched Canon EXACTLY** before the fix under the resolve rule (first non-zero of
  `ProductBOM.UnitPrice` then `PartPrices`) — independent validation that the rule is right.
- ⚠️ **3 BOM rows remain priced 0 ON PURPOSE — do not "fix" them.** L307 BOMID 2549 and L313 BOMID
  2802 each have a **priced `Full` row for the same part**; the zero on the feeder row is the guard
  that stops the bare board being counted twice. (L254 BOMID 2390 `YH4-3057-000` is a real gap but
  L254 is obsolete — left alone.)

**🔴 STILL OPEN — L261's two line copies disagree on QUANTITY (not price):** after the fix Line 1 =
RM 23.21/board vs Line 3 = RM 20.75. Five connectors are **qty 4 on Line 1 vs 1 on Line 3**
(`VS1-8550-005`, `-8550-007`, `-8887-012`, `-9243-007`, `-9243-008`) plus `WA7-8586-000` (1 vs 2).
Line 3 is the copy that reconciles to the verified RM 26.55/board. **Not changed** — quantities also
drive what PVS expects at each feeder, so it needs Danial's decision. See [[canon-board-costing]].

**🎁 FOC (FREE-OF-CHARGE) PARTS EXIST — Danial 2026-08-01: "sometimes Canon ship up parts for new
model test case and no invoice issued, parts are on FOC basis."** So **a receipt with no invoice is
LEGITIMATE, not a missing document** — never chase it as a discrepancy.
- Fingerprint seen live: **16 Canon-format parts, 88,340 pcs, unpriced, in NO BOM**, on hand with
  `RemainingQty == Qty` (never issued), **15 of the 16 stocked in on the same day 14-05-2026, one reel
  each**, and absent from all 1,054 Mar–Jul invoice lines. Their part families shadow production parts
  with different suffixes (`VS1-9540-006` vs invoiced -004/-005/-007/-009/-012; `VS1-9270-020` vs
  -008/-010/-012/-014) ⇒ a new-model variant trial. List:
  `Downloads\canon-unpriced-stock-16-parts.csv`.
- ⚠️ **I twice got the valuation logic BACKWARDS — do not repeat it.** I said unpriced stock was
  "money missing from the valuation"; for FOC parts the opposite holds — **they cost GMS nothing, so
  pricing them at Canon's rate INFLATES inventory with material never paid for.** FOC dead stock is a
  **space/obsolescence** problem, not a cash one. (I also wrongly said these parts "predate the
  invoices" — they were booked in May, inside the invoice range.)
- ⇒ **MCS DESIGN REQUIREMENT: every receipt carries a COST BASIS (purchased vs FOC), not just a
  quantity.** Otherwise valuation inflates, FOC receipts look like missing invoices, and loss
  reporting charges GMS for material it never bought. This is a **fourth** reconciliation state
  alongside received/consumed/on-hand.
- Useful question for these 16 is **which model/trial they were shipped for and whether it is still
  live** — NOT "is there an earlier invoice" (there won't be one).

**Other invoice files (not yet parsed):** `Downloads\canon invoice\Invoice\` holds 12 PDFs — mostly
**GMS→Canon** outbound invoices + DOs, plus `SC DN 260620-002`, `SC-DN-260608-009`, `HC-26L0004`.
Only `Part Invoice\*.pdf` (Mar–Jul) has been OCR'd.

Related: [[mcs-material-control-system]], [[parts-control-db-write-access]].
