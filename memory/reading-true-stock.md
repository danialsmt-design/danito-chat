---
name: reading-true-stock
description: StockIns.RemainingQty is ZEROED on issue — store-only stock reads zero for anything on the feeders.
metadata: 
  node_type: memory
  type: project
  originSessionId: 45a3a5d4-9c33-4440-ac3d-328171bfb409
  modified: 2026-08-07T12:25:46.078Z
---

To answer "how much of part X do we have", store stock alone is **wrong, not merely incomplete**:

```
have(part) = SUM(StockIns.RemainingQty)                     -- store, dedupe by UID (no identity column,
                                                            --   6 UIDs carry duplicate live rows -> use MAX)
           + SUM(latest StockOuts row per PartUID)           -- live at the lines, ORDER BY ID DESC
             bounded to reels issued in the last ~30 days
```

Three traps, all verified live on 7 Aug 2026:

1. **`StockIns.RemainingQty` is set to 0 when a reel is issued**, not frozen at its value. So a part sitting
   on the feeders reads as *zero in store*. Real examples: `VV5-3355-103` store 0 / line 247,944;
   `VE3-6800-105` store 0 / line 197,296. Reading store-only made 26 of L254's 28 parts look exhausted and
   produced a fictitious RM 13,754 shortage; the true figure was RM 305 across 2 parts.
2. **Take only the LATEST StockOuts row per `PartUID` (`ORDER BY ID DESC`).** Summing all rows for a UID
   double-counts refills — `WA6-3110-000` under placeholder UID `0000-A830%` has nine rows summing to
   10,135 when the truth is 338.
3. **A date bound is mandatory.** StockOuts keeps a row per reel forever, and reels consumed before PVS began
   writing balances back still show their issued quantity. Unbounded, the plant total reads **30.7 million
   pieces** and no shortage can ever surface; bounded to 30 days, 7.8 million.

Coverage caveat: only lines that run `syncStockOuts` keep those balances live (Lines 1, 2, 5 as of Aug 2026 —
see [[pvs-line2-deploy]], [[pvs-line5-deploy]]). On Lines 3 and 4 the figure is the reel's *issued* quantity,
so a shortage there can be **bigger** than calculated, never smaller.

Also: a two-sided lot logs its boards in DailyProductionCount under BOTH sides, so summing says a 300-board
lot built 600. Take the least-progressed side.

**Why:** these all fail in the direction that *hides* problems or *invents* them, and both directions reach
the customer as a wrong parts request.

**How to apply:** never quote available stock, a shortage, or "we have none left" from `StockIns` alone.
Pair with [[canon-bom-source-of-truth]] — a stock figure is only as good as the per-board quantity it is
compared against.
