# LOB-SIM — Limit Order Book Manipulation Simulator

**A browser-based, single-file simulation of an E-mini S&P 500 futures market with realistic agent behaviour, six manipulation modes, forensic analysis, and academic references.**

Prepared by Stanley Yong · Graduate Finance / Compliance Programme  
Part of the *Trade Surveillance: Traditional Techniques, Rogue Trading & Market Abuse* lecture series.

---

## What this is

LOB-SIM is an interactive teaching tool that lets you watch market manipulation happen in a simulated limit order book — and then analyse its forensic signature. It is designed to make abstract concepts (spoofing, front-running, marking the close) concrete and visible. Everything runs in a single HTML file in any modern browser with no server, no dependencies, and no installation.

The simulation is built on:

- A 10-level limit order book for **E-mini S&P 500 futures (ES·Z25)**, using realistic CME contract specifications ($0.25 tick, $12.50/tick value, $11,500 initial margin)
- **12 distinct agents** with independently calibrated decision logic, position limits, fees, and capital constraints
- **6 manipulation modes** that can be switched at any time during a live simulation
- A **forensics tab** showing the statistical signatures regulators use to detect each manipulation type
- **Downloadable CSV exports** for further analysis
- **Academic references** throughout — every metric in the forensics panel links back to the paper that established it

---

## Quick start

1. Open `lob_sim.html` in Chrome, Firefox, Safari, or Edge (any modern browser)
2. Select a manipulation mode from the selector bar — start with **⚡ Spoofing** to see the most documented form
3. Click **▶ START**
4. Watch the order book, time & sales tape, and event log
5. Switch to the **⚖ Forensics** tab to see the statistical analysis build in real time
6. Click any **agent card** to read about that agent's decision rules and see its live state
7. Click **📖 Why these metrics?** to read the academic basis for each forensic signal

---

## Instrument specification

| Parameter | Value |
|---|---|
| Instrument | E-mini S&P 500 Futures — ES·Z25 |
| Tick size | $0.25 per index point |
| Tick value | $12.50 per contract |
| Contract multiplier | $50 × index |
| Initial margin (CME) | $11,500 per contract |
| Maintenance margin | $10,500 per contract |
| Order book depth | 10 levels each side |
| Taker fee | $1.00 per contract |
| Maker rebate | −$0.25 per contract |
| Trading day | 06:00–16:00 CT (simulated) |

---

## Interface layout

```
┌─────────────────────────────────────────────────────────────┐
│  TOP BAR: Last price · High/Low · Volume · VWAP · Speed     │
├─────────────────────────────────────────────────────────────┤
│  MANIPULATION SELECTOR + contract spec                      │
├─────────────────────────────────────────────────────────────┤
│  DAY PROGRESS BAR (session markers: Open / Mid / Close)     │
├─────────────────────────────────────────────────────────────┤
│  SKIP BAR: session jumps + ±30m/1h buttons + scrubber       │
├──────────┬──────────┬──────────────────┬────────────────────┤
│ ORDER    │ AGENT    │ PRICE CHART      │ TIME & SALES       │
│ BOOK     │ PANEL    │ (intraday)       │ ── or ──           │
│ (10 lvl) │ (cards)  │                  │ FORENSICS          │
│          │          │                  │ [📖 Why metrics?]  │
├──────────┴──────────┴──────────────────┴────────────────────┤
│  EVENT LOG — information flow & surveillance signals        │
└─────────────────────────────────────────────────────────────┘
```

### Order book panel

Shows the 10 best bid and ask price levels, with depth bars scaled to the largest resting quantity visible. Orders from manipulation agents are marked with a **●** dot coloured by agent. The spread and mid-price are shown between the two sides.

### Agent positioning panel

One card per active agent, showing:

- **Name and type** (click to open the agent detail modal)
- **Capital bar** (retail/noise agents only): current available capital as a percentage of starting capital, with a purple overlay showing how much is committed as margin
- **Bid/ask exposure bar**: current resting quantity on each side
- **P&L** (realised + mark-to-mid unrealised), **Fills**, **Net position**, **Cancel rate**

When a retail agent goes bankrupt the card dims, shows "BANKRUPT", and counts down to rebirth.

### Price chart

Intraday price history with volume bars. Session bands (pre-market / open / mid / close) are shaded. The settlement window (15:45+) is highlighted in purple in the close session. Vertical dashed lines mark session transitions.

### Time & Sales tape

Every fill printed to the tape with timestamp, price, quantity, and an agent-type tag (MANIP / FUND / MM / WASH / INSIDE / FRONT). Rows are taller and readable at normal browser zoom.

### Forensics tab

Mode-aware statistical analysis that updates every 3 seconds. Content changes depending on the active manipulation mode — see the [Manipulation modes](#manipulation-modes) section below for what each forensics view shows. The **📖 Why these metrics?** button opens a modal with the academic basis for every metric displayed.

### Event log

Chronological feed of simulation events. Colour-coded by type:

- **White**: general market commentary
- **Pink (ℹ INFO LEAK)**: private information visible only to the insider agent — not public
- **Cyan (⬆ FLOW)**: large order flow events
- **Purple (◆ CLOSE)**: close session and settlement window events  
- **Red (⚠ SIGNAL)**: manipulation activity and surveillance signals

---

## Controls

### Speed

Use the **1× / 3× / 8× / 20×** buttons in the top bar. At 20× a full trading day runs in approximately 3 minutes. The simulation updates at up to 20 sim-seconds per real second at high speeds.

### Session jump buttons

The skip bar provides five preset jump points:

| Button | Wall time | Purpose |
|---|---|---|
| **06:00 Pre** | 06:00 CT | Pre-market — wide spreads, thin book |
| **09:39 Open** | 09:39 CT | Just past the open — highest volatility |
| **12:00 Mid** | 12:00 CT | Mid-session — tightest spreads |
| **15:00 Close** | 15:00 CT | Close session activates — marking strategy begins |
| **15:55 Mark** | 15:55 CT | Settlement window — most aggressive marking activity |

The **−1h / −30m / +30m / +1h** buttons move relative to the current position. Rewinding resets state and fast-forwards silently to the target time.

### Scrubber

Click anywhere on the gradient bar to jump to that point in the trading day. Hover to preview the timestamp and session name before committing.

---

## Manipulation modes

Switch modes at any time using the selector bar. The simulation adapts immediately — manipulation agents activate or deactivate, and the forensics panel updates its content.

### ⚡ Spoofing

**What happens:** Agents MN1 and MN2 place large visible orders (200–400 lots) 3–6 ticks deep on one side of the book with no intention of filling them. A small genuine order (5–15 lots) is simultaneously placed on the opposite side at best price. The spoof orders create artificial depth that attracts other participants; the genuine order fills in 2–6 sim-seconds, then the spoof orders cancel.

**Decision logic:** MN1 only places a spoof episode when: (1) the spread is at least 1 tick (sufficient room to extract edge), (2) cumulative P&L is above −$2,000, and (3) the book imbalance favours the chosen direction. Spoof side is chosen based on which side already has more resting depth — reinforcing it creates a larger visible distortion.

**Forensics shows:** Cancel ratio (target >70%), order size vs book depth ratio, price impact per episode, the place→fill→cancel sequence log.

**Key reference:** Attari, Lynch, Chhabra & Fazilet (2020). *Primer on Futures Markets and Spoofing Allegations.* CRA Insights.

---

### 📚 Layering

**What happens:** MN1 stacks 4–7 orders at successive price levels on one side, creating an artificial wall of apparent supply or demand. MN2 places the genuine execution order on the opposite (thin) side. All layers cancel simultaneously after the genuine order fills — the defining forensic signature.

**How it differs from spoofing:** Multiple levels vs one large order. The depth distortion is more persistent and distributed. Harder to detect with single-threshold rules; requires multi-level sequence detection.

**Forensics shows:** Layers placed vs removed, the simultaneous cancellation pattern, MN1/MN2 resting exposure by side.

**Key reference:** FCA Final Notice: Michael Coscia (2013) — first European layering prosecution.

---

### 🔔 Marking the Close

**What happens:** MN1 and MN2 are inactive until the CLOSE session (15:00 CT). In the early close phase (15:00–15:45) MN1 builds bid support. In the settlement window (15:45+) it aggressively lifts offers in 50–140 lot increments, pushing the last traded price (the settlement reference) artificially higher. Most visible on the price chart as an uptick in the final minutes with no fundamental catalyst.

**Why it matters:** Many derivatives, ETFs, and index products are valued at the settlement price. Pushing settlement a few ticks can generate disproportionate gains on derivative positions.

**Jump to see it:** Use **15:55 Mark** to skip directly to the settlement window. The forensics panel shows the Close/Mid volume rate ratio (anomalous above 1.5×) and MN1's share of settlement-window volume.

**Key reference:** Comerton-Forde & Putniņš (2011). *Measuring Closing Price Manipulation.* Journal of Financial Intermediation 20(2).

---

### 🏃 Front-Running

**What happens:** MN1 receives advance information about Blackrock's (FD2's) forthcoming large order — shown in the event log as a **pink INFO LEAK** entry, representing a private communication channel invisible to all other participants. One sim-second before FD2 executes, MN1 builds a position in the same direction. After FD2's large order moves the price, MN1 closes into the impact for a profit.

**What to watch:** The pink INFO LEAK appears first. Then MN1's fill. Then FD2's large order. Then MN1's close. The time delta between the leak and the public order is the information advantage window.

**Key reference:** US v. Johnson (2017) — first criminal conviction of a senior banker for front-running a client FX order. Key evidence: recorded phone call.

---

### 🔄 Wash Trading

**What happens:** WS1 places matching buy and sell orders at the same price, crossing them with itself. Both sides of the trade belong to WS1 — no change in beneficial ownership, no transfer of risk. Volume statistics are inflated. The session has a volume target (800–2,000 lots) and a fee budget ($1,500); WS1 stops when either is exhausted.

**Detection limit:** This manipulation is **invisible from the public tape alone**. A wash trade looks identical to a genuine trade. Detection requires cross-account beneficial ownership analysis at the exchange or regulatory level — unavailable from market data.

**Key reference:** CFTC v. Coinbase (2021); SEC v. Lek Securities Corp (2017).

---

### 🔍 Insider Trading

**What happens:** After approximately 30 simulation minutes, agent IN1 receives a **private information event** (e.g. a forthcoming Fed signal) — shown in pink in the event log and invisible to all other participants. IN1 quietly accumulates a position in the anticipated direction. When the news breaks publicly (10–20 sim-minutes later), all participants see the announcement simultaneously, the price moves, and IN1 exits for a profit.

**The forensic challenge:** IN1's pre-announcement accumulation looks like a patient directional trader. The pattern is circumstantially suspicious but not conclusive — a skilled analyst could arrive at the same position through legitimate research. Prosecution requires establishing the information chain: who knew, when, and how it was communicated.

**Key reference:** Meulbroek (1992). *An Empirical Analysis of Illegal Insider Trading.* Journal of Finance 47(5).

---

## Agent roster

| Agent | Name | Type | Capital | Key behaviour |
|---|---|---|---|---|
| MM1 | JPM-MM | Market Maker | Institutional | Adaptive spread quoting, adverse selection tracking |
| MM2 | GS-MM | Market Maker | Institutional | Independent market maker, same architecture as MM1 |
| NT1 | Retail-A | Noise Trader | $15k–$50k | Bull/bear signal + momentum, 4-tick stop-loss, 20-contract limit |
| NT2 | Retail-B | Noise Trader | $15k–$50k | More risk-averse, 3-tick stop-loss, 15-contract limit |
| NT3 | Retail-C | Noise Trader | $15k–$50k | Widest stops (5 ticks), largest limit (25 contracts) |
| FD1 | Vanguard | Buy-Side Fund | Institutional | TWAP mandate, VWAP-disciplined execution |
| FD2 | Blackrock | Buy-Side Fund | Institutional | TWAP mandate, front-running target |
| AR1 | SIG-Arb | Arbitrageur | Institutional | VWAP mean-reversion, +$150 take-profit, −$300 stop |
| MN1 | ???-Alpha | Manipulator | N/A | Primary manipulation agent (mode-dependent) |
| MN2 | ???-Beta | Manipulator | N/A | Support agent, coordinates with MN1 |
| WS1 | ???-Wash | Wash Trader | N/A | Volume inflation via self-crossing |
| IN1 | ???-Priv | Insider | N/A | Trades on material non-public information |

Click any agent card header to open its detail modal with full decision rules, live strategy state, and academic references.

---

## Capital and margin model

Retail agents (NT1–NT3) have finite capital. This creates realistic constraints and eventual account wipeout.

**Starting capital:** $15,000–$50,000, randomised at simulation start.

**Position limit:** Derived from capital — `floor(capital / $11,500)`. A $15k account can hold at most 1 ES contract; a $45k account can hold 3.

**Margin call:** When total equity (cash + unrealised P&L) falls below `|position| × $10,500`, the broker force-closes the entire position. The agent survives but with a smaller capital base and a tighter position limit.

**Bankruptcy:** When total equity reaches zero (losses exceed starting capital), the account is wiped out. The agent card dims and shows "BANKRUPT" with a countdown to rebirth (60–180 sim-seconds).

**Rebirth:** A new retail participant enters with fresh randomised capital, a new directional bias, and a generation suffix on their name (Retail-A², Retail-A³, etc.). The panel badge tracks cumulative bankruptcies.

---

## Fees

| Transaction type | Fee |
|---|---|
| Taker (aggressive/market order) | $1.00 per contract |
| Maker (passive limit order, filled) | −$0.25 per contract (rebate) |
| Wash trade (both sides) | $2.00 per contract per side |

Fees are applied immediately on every fill and flow into realised P&L. Over a session, fee drag is material for high-frequency noise traders and wash traders.

---

## Data exports

Three CSV download buttons are available in the interface:

| Button | Content |
|---|---|
| **⬇ T&S CSV** | Every fill: time, price, quantity, buyer/seller agent, tag, session, mode |
| **⬇ SIM CSV** | Price history (every 10 sim-seconds) + agent summary + session volume breakdown |
| **⬇ EVENT LOG CSV** | Every event log entry: time, type, session, mode, full text |

All exports are named by mode and simulation time, e.g. `ES_Z25_sim_spoofing_12h30.csv`.

---

## Forensics reference

The forensics tab shows mode-specific metrics. Click **📖 Why these metrics?** to open the research modal with full academic context. Key signals by mode:

| Mode | Primary forensic signal | Threshold | Source |
|---|---|---|---|
| Spoofing | Cancel rate for large orders | >70% | CFTC v. Sarao (2015); CRA (2020) |
| Layering | Place→fill-opposite→cancel sequence | Simultaneous cancel of all layers | FCA v. Coscia (2013) |
| Marking Close | Close/Mid volume rate ratio | >1.5× is anomalous | Comerton-Forde & Putniņš (2011) |
| Marking Close | Single-agent settlement window share | >15% is anomalous | Putniņš (2012) |
| Front-Running | MN1 fill timing vs FD2 execution | <3s delta | FCA/HSBC (2016) |
| Wash Trading | Beneficial ownership overlap | Any self-cross | SEC v. Lek (2017) |
| Insider Trading | Pre-announcement accumulation | 40–50% of move precedes news | Meulbroek (1992) |

---

## Technical notes

**No installation required.** The simulation is a single self-contained HTML file. Open it in any modern browser. No server, no npm, no Python environment.

**Performance.** At 20× speed the simulation processes up to 300 sim-steps per animation frame during skip operations. The price chart redraws every 2 sim-seconds. Agent cards update on every step. On older hardware, use 8× or lower for smooth rendering.

**Browser compatibility.** Tested on Chrome 120+, Firefox 121+, Safari 17+. Requires ES2020 JavaScript support. The file uses no external libraries — all rendering is native Canvas and DOM.

**File size.** The complete simulator is approximately 3,800 lines of HTML/CSS/JavaScript in a single file (~220KB uncompressed).

---

## Pedagogical context

This simulator supports the *Trade Surveillance* lecture module, which sits between Enterprise Risk Management and RegTech in the curriculum. It is designed to be used:

1. **Before the lecture** — students can explore the tool freely to build intuition about order book mechanics
2. **During the lecture** — the instructor can run specific modes live to illustrate each manipulation type as it is discussed
3. **After the lecture** — students can export CSV data and run their own detection algorithm experiments

The simulation is calibrated to produce realistic volume shares for manipulation agents (typically 5–15% of total volume during active manipulation episodes, spiking to 20–30% during concentrated events), realistic P&L trajectories for each agent type, and forensic signatures that match the empirical literature.

---

## Primary sources

| Source | Used for |
|---|---|
| Attari, Lynch, Chhabra & Fazilet (2020). CRA Insights: Finance | Spoofing mechanics, Flaum case reconstruction |
| Comerton-Forde & Putniņš (2011). Journal of Financial Intermediation | Marking the close — Close/Mid ratio |
| Meulbroek (1992). Journal of Finance | Insider trading — pre-announcement signal |
| Avellaneda & Stoikov (2008). Quantitative Finance | Market maker adaptive spread model |
| Almgren & Chriss (2001). Journal of Risk | Institutional TWAP execution |
| Kyle (1985). Econometrica | Noise trader theory |
| Glosten & Milgrom (1985). Journal of Financial Economics | Bid-ask spread and adverse selection |
| CFTC Orders 19-15, 19-16 (2019) | Flaum, Edmonds spoofing cases |
| DOJ v. JPMorgan (2020) | Institutional spoofing, RICO application |
| FCA v. Coscia (2013) | Layering — first European prosecution |
| US v. Johnson / FCA v. HSBC (2016–17) | Front-running, comms evidence |
| PP v. Soh Chee Wen [2023] SGHC 299 | Singapore BAL manipulation case |
| EU Regulation 596/2014 (MAR) | Market abuse definitions |
| MAS-SGX Trade Surveillance Practice Guide (2019) | Singapore three-tier surveillance |

---

*Prepared by Stanley Yong · Current Market Developments Extension Module · Graduate Finance / Compliance Programme*
