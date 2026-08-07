---
name: bursa-trading-agent
description: "Bursa day-trading agent experiment — measured cost floor of 0.469% per round trip, and why buy-and-hold beat every backtested variant."
metadata: 
  node_type: memory
  type: project
  originSessionId: 7e18fc24-29fb-4463-a95e-e6dd30466813
  modified: 2026-08-07T01:41:21.469Z
---

Danial is exploring an automated Bursa Malaysia research/trading agent, delivered
daily over WhatsApp (see [[gmail-whatsapp-digest]] for the same bridge). Work done
7 Aug 2026; scripts lived in scratchpad and are gone, but these findings stand.

**Measured Malaysian retail round-trip cost: 0.469% of notional.**
Brokerage 0.1% (min RM8) + clearing 0.03% + stamp duty 0.1%, charged both legs.
This is the binding constraint on any intraday strategy — with ~3% average daily
range on liquid counters, a trade must capture ~16% of the whole day's range just
to break even.

**Backtest results (5 liquid counters, 120 sessions, RM50k, momentum rule:
above 20d MA + RSI<70 + prior day green, long only):**
- Intraday open→close: gross +5%, fees 13.81% of capital, **net −8.72%**
- Holding 20 days: fee drag falls to 2.7%, **net +37.14%**
- **Buy & hold the same 5 stocks: +54.35%** — beat every variant tested
- Returns across holds were non-monotonic (+4.4/−4.0/+29.3/+16.0/+24.2/+37.1%),
  and the zero-cost control jumped identically → the instability is in the
  *signal*, not the costs. Small sample (21–123 trades), and the window was a
  strong bull run, so it flatters any long-only rule.

**Conclusion reached: build the research/alerting brief, not a signal generator.**

**Data access (researched, high confidence):** Bursa sells no retail data product;
entry is RM11,600/mo distributor licence. Real-time is pointless for a pre-open
brief — at 7:30am delayed and real-time data are identical. TradingView RM9/mo and
KLSE Screener RM5/mo licence data **display-only** and prohibit scripting. Bursa's
Non-Display Usage regime (RM580/mo "investment analysis", RM5,800/mo automated
trading) nominally covers scripted use of *delayed and EOD data too*. Clean paid
option if ever needed: EODHD $19.99/mo. Free path: Yahoo's JSON chart endpoint
`query1.finance.yahoo.com/v8/finance/chart/<code>.KL?range=1y&interval=1d` —
works where the HTML pages 403, gives full OHLCV to compute indicators locally
rather than scraping them. Violates Yahoo ToS §2.4(i); low practical risk at one
request/day, but it is a violation, not a blessed use.

**Delivery constraint:** Claude Code scheduled tasks only fire while the app is
open — no good with the desktop off. Danial raised routing through a Raspberry Pi,
which is the right fix since the WhatsApp bridge is local anyway.
