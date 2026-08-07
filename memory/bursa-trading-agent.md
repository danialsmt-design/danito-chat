---
name: bursa-trading-agent
description: "Bursa day-trading agent experiment — measured cost floor of 0.469% per round trip, and why buy-and-hold beat every backtested variant."
metadata: 
  node_type: memory
  type: project
  originSessionId: 7e18fc24-29fb-4463-a95e-e6dd30466813
  modified: 2026-08-07T02:25:32.592Z
---

Danial is exploring an automated Bursa Malaysia research/trading agent, delivered
daily over WhatsApp (see [[gmail-whatsapp-digest]] for the same bridge). Work done
7 Aug 2026; scripts lived in scratchpad and are gone, but these findings stand.

**Malaysian retail round-trip cost: 0.469% of notional is a FLOOR, not a constant.**
Brokerage 0.1% (min RM8) + clearing 0.03% + stamp duty 0.1%, both legs. The RM8
minimum binds below ~RM8,000 notional: a RM3,000 round trip costs **0.793%**.
Also **add 8% SST on commission + clearing** (Malaysia, since 1 Oct 2025) — the
first model missed it. Note 0.1%/RM8 is a traditional-broker convention, not a
Bursa rule; Moomoo MY charges 0.03% + RM3/order, Rakuten is flat-tiered from RM1.
Either way this is the binding constraint intraday — with ~3% average daily range,
a trade must capture roughly a sixth of the whole day's range just to break even.

**Backtest results (5 liquid counters, 120 sessions, RM50k, momentum rule:
above 20d MA + RSI<70 + prior day green, long only):**
- Intraday open→close: gross +5%, fees 13.81% of capital, **net −8.72%**
- Holding 20 days: fee drag falls to 2.7%, **net +37.14%**
- **Buy & hold the same 5 stocks: +54.35%** — beat every variant tested
- Returns across holds were non-monotonic (+4.4/−4.0/+29.3/+16.0/+24.2/+37.1%),
  and the zero-cost control jumped identically → the instability is in the
  *signal*, not the costs. Small sample (21–123 trades), and the window was a
  strong bull run, so it flatters any long-only rule.

Two flaws found later, both making the result *worse* than measured: small trades
were under-costed (RM8 floor above), and **Yahoo 404s delisted `.KL` tickers**
(MAS 3786, UMW 4588 both gone) so the backtest only ever saw survivors. Bursa's
M10 "IPOs and Delisted Companies" package (RM1,000/yr) is the only fix found.

**Conclusion reached: build the research/alerting brief, not a signal generator.**
Independent survey (7 Aug 2026) backs this hard: of 77 LLM-trading studies, only
1 of 19 closed-loop ones modelled transaction costs and none were reproducible;
an independent replication of the leading agents over 20 years found buy-and-hold
beat them with no significant alpha (p > 0.34); and in the one real-money test
(6 frontier LLMs, $10k each) four of six lost with **fees dominating PnL** — the
same failure mode measured here. TradingAgents (96k stars, claimed Sharpe 8.2)
models no costs and has a maintainer-acknowledged look-ahead bug (issue #203).
Don't screen on attention/unusual volume: retail order flow returns −14.8%
annualized in high-attention stocks vs +6.6% elsewhere — the RVOL screen built
here steers into the bad bucket.

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

**Broker API — corrects an earlier wrong claim here.** It is NOT true that no
Malaysian broker exposes an API: **Moomoo Securities Malaysia has an order-execution
API** for Bursa equities (Python/Java/C# over a local OpenD gateway, no extra fee),
and is the only entity in the Futu/moomoo group that can trade Malaysia via API.
Caveats: **no paper trading for the Malaysian market**, and moomoo's own docs
contradict each other on whether the API serves Bursa *quotes* (intro page says
yes, Authorities page says Unsupported). Unresolved by email to
support@my.moomoo.com. IBKR carries Bursa but multiple secondary sources say
equities are barred to Malaysian residents (futures aren't) — unverified.

**Delivery constraint:** Claude Code scheduled tasks only fire while the app is
open — no good with the desktop off. Danial raised routing through a Raspberry Pi,
which is the right fix since the WhatsApp bridge is local anyway.
