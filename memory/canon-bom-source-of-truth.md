---
name: canon-bom-source-of-truth
description: "ProductBOM.Quantity is NOT a reliable per-board mount count — Canon's MSPC PDFs are; how to read them."
metadata: 
  node_type: memory
  type: project
  originSessionId: 45a3a5d4-9c33-4440-ac3d-328171bfb409
  modified: 2026-08-07T12:15:53.923Z
---

`ProductBOM.Quantity` in ReelPart-New is **not** a trustworthy per-board mount count. Proven 7 Aug 2026:
L254 stored `WA6-3110-000` at **31 per board** when Canon's MSPC says **1** (`WA6-2368-000`: DB 5, Canon 1).
At 520 boards that is a 16,120-piece purchase request instead of 520. PVS's own `ReelPartContracts.cs`
already warns the field is "for reference only" and that the forecast uses the machine feeder-list Mount
Step instead — so this was a documented trap, not a surprise.

**The authority is Canon's MSPC documents at `D:\CANON BOM LIST\`** (one folder per model), section
"D : PARTS LIST", columns PART NO / PART NAME / QTY / LEVEL.

Reading them:
- **The PDFs are password-protected. The password is the model number itself** (`L261`, `L307`, `L311`, …).
- **`pypdf` default `extract_text()` jumbles the QTY column into an unreadable digit blob** — e.g.
  `"1 11 11 11 11 12 11..."`. Hand-aligning that blob is how a wrong quantity gets invented. Always use
  `extract_text(extraction_mode="layout")`, which aligns the columns and makes QTY unambiguous.
- The QTY digit is usually **glued to the end of the maker/type text** (`...RP130N301D@-FE1  1` = qty 1,
  level 1). Trailing pair is QTY then LEVEL.
- **LEVEL 2 rows are alternates/substitutes** for the line above (often bracketed, marked "Choice") — not
  separate parts. Counting them as parts inflates the BOM. See [[pvs-substitute-parts]].
- A part legitimately occupying several feeder positions has several ProductBOM rows, each with its own
  placement count, so the DB per-board total is a SUM over rows — that part is *not* a bug.

Known gaps as of 7 Aug 2026: **L347 has a Canon MSPC but zero ProductBOM rows**; **L309 has BOM rows but no
MSPC in the folder**; **L254 (ProductID 0) has 53 rows and not one supply position**, so nothing in its BOM
was ever checked against a machine feeder list. `BomTrust` in `Pvs.Core/Inventory/` encodes these checks.

**Why:** every shortage, cost and reorder figure is derived from this field, so a single bad quantity becomes
a purchase request to the customer.

**How to apply:** never quote a per-board quantity, shortage or material cost from `ProductBOM.Quantity`
alone — check it against the MSPC first, or state plainly that it is unverified. See
[[canon-board-costing]] and [[canon-part-invoices]].
