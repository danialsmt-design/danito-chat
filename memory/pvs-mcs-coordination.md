---
name: pvs-mcs-coordination
description: "PVS and MCS Ai share one DB — the cross-session coordination ledger, ownership map, and single-writer rule that keep them from contradicting."
metadata: 
  node_type: memory
  type: project
  originSessionId: 19fcfab6-cf20-4ed1-99bb-c8d8f04306be
  modified: 2026-08-14T13:33:28.513Z
---

Two workstreams write the SAME database `ReelPart-New` and must not contradict (Danial 2026-08-14: "real need for coordination so the workspaces work in synergy"): **PVS** (this session — line runtime, feeders, reel retire/attrition, counts, stale-stock reconciliation) and **MCS Ai** (the [[mcs-ai-agent-spec]] session — new/rehosted DB + data-entry pages + 24/7 guardian on the NAS).

**The two sessions live in different project folders, so their memory does NOT cross.** The coordination channel is a neutral shared file both read/append: **`Documents/Dantec/MCS/COORDINATION.md`** (the ledger). The build handoff is `Documents/Dantec/MCS/MCS-Ai-BUILD-BRIEF.md`.

**Rule (from CLAUDE.md concurrency/authority):** one writer per resource, one integration owner, claim-before-write.
- **Ownership:** MCS Ai owns **schema/DDL + entry** (StockIns, StockOut issue INSERTs, PartRanks/ProductBOM/price master). PVS owns **runtime updates** (StockOut qty decrement + retire→0, ConsumedReels, PartAttrition, production counts). Split of the one shared table `StockOuts` is **by operation**: MCS Ai INSERTs issues, PVS UPDATEs qty on existing reels; neither deletes the other's rows.
- **Schema decision (2026-08-14, Danial said "you decide"):** MCS Ai = single schema system of record. Transition rule until NAS-DB cutover: PVS may apply **additive-only** DDL (new tables/columns, never alter/drop/rename) on the current DB, and **must log each in the ledger's Schema-change log** so MCS Ai folds it into the new-DB schema (no drift at cutover). Non-additive → MCS Ai owns, PVS files a request, Danial approves. After cutover: all DDL through MCS Ai.
- **Integration owner = Danial:** all DDL, bulk/destructive writes, and the DB→NAS cutover need his explicit go.
- **Protocol:** announce intent in the ledger → wait for approval if DDL/bulk/destructive → other side stands off → transaction+CSV-backup so it's reversible → release the claim.
- **Cutover** ([[mcs-cutover-readiness]], [[parts-db-server-migration]]) is a single coordinated window: MCS Ai stands up the NAS host, PVS repoints all writers, Danial calls the cut — never independently.

**Three writers zero `StockOuts.Quantity` today** (all auditable via `LastQuantityChangeDate` — every zeroing is stamped; batch list-clear = many rows on one ms timestamp; auto-retire = UID in `ConsumedReels`): (a) PVS app auto-retire on reel swap; (b) **Ashish's manual scan/clear HTML page** — operators return empty reels → he scans & zeroes, plus batch "clear this list" of a verified list (14 Aug he batch-cleared 4,275 reels in two shots + 58 individual); (c) my bulk reconciliation ([[stockout-stale-reconciliation]], backed up). **MCS Ai's returns/stock-out-clear pages = the SAME job as Ashish's page → must replace it, not run alongside** (Danial decides that function's cutover). Accuracy risk: a stale list-clear can zero reels still loaded on a feeder — cross-check cleared UIDs vs `/api/exhaust`.

Before any PVS change that touches a shared table or schema, check the ledger first.
