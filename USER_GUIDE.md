# LOB-SIM User Guide

**Getting started with the Limit Order Book Manipulation Simulator**

---

## Before you begin

You need one thing: the file `lob_sim.html`. Open it in Chrome, Firefox, or Safari. Nothing else installs. No account, no internet connection required once the file is open.

If the page loads blank, your browser may be blocking local HTML files. Try right-clicking the file and selecting *Open with → Chrome* (or your browser of choice). If you're on a corporate machine, try opening it from your Downloads folder rather than a network drive.

---

## First thirty seconds

When the page opens you'll see a dark terminal-style interface with no data yet. Here is the fastest path to seeing something meaningful:

1. In the **Manipulation Mode** bar near the top, click **⚡ Spoofing**
2. A brief explanation will appear — read it, then dismiss it
3. Click **▶ START** in the top-right area of the header
4. Click **8×** to set the speed to 8×
5. Watch the order book on the left fill with price levels and quantities

Within a few seconds you'll see the Time & Sales tape (right column) printing fills, the price chart building, and the event log at the bottom showing activity.

After about 30 real seconds at 8× speed the simulation will reach the main trading session and you'll start seeing spoof-related entries in the event log in red.

---

## Reading the order book

The order book is the tall panel on the far left. It shows the 10 best prices at which someone is willing to buy (bids, green, bottom half) and the 10 best prices at which someone is willing to sell (asks/offers, red, top half).

```
QTY        PRICE       QTY
           5893.00     42    ← ask: sellers here
           5892.75     18
           5892.50     31
─────────── SPREAD ──────────
32         5892.25           ← bid: buyers here
61         5892.00
14         5891.75
```

The **spread** is the gap between the best ask and best bid — here $0.25 (1 tick). This is the cost of an immediate trade: if you want to buy right now you pay the ask; if you want to sell right now you accept the bid.

**Depth bars** — the coloured horizontal bars behind each row — show relative quantity. A very long green bar means a large resting buy order at that level.

**Manipulation markers** — when a manipulation mode is active, a small **●** dot appears on order rows placed by manipulation agents. The colour matches the agent type.

---

## Reading the agent panel

The second column from the left shows one card for each active market participant.

Each card has:

- **Header bar** (click to read about that agent's rules and see live stats)
- **Capital bar** — for retail traders only: the green/amber/red bar shows how much of their starting capital remains. Purple overlay shows how much is tied up in margin
- **Bid/Ask exposure bar** — how many contracts they currently have resting on each side of the book
- **P&L** — their total profit or loss since the simulation started
- **Fills** — total contracts traded
- **Pos** — their current net position (+ = long, − = short)
- **CxlR** — cancel rate (for manipulation agents, this goes very high)

**If you see "BANKRUPT"** on a retail agent card, that trader has lost their entire account. They'll re-enter the market in 1–3 simulation minutes as a new account (Retail-A², Retail-B³, etc.).

---

## The session timeline

The thin progress bar below the manipulation selector shows where you are in the trading day. Four sessions run left to right:

| Session | Time | What to expect |
|---|---|---|
| Pre-market | 06:00–09:30 | Wide spreads, thin book, no manipulation |
| Open | 09:30–11:00 | Highest volatility, most activity |
| Mid-session | 11:00–15:00 | Calm, tightest spreads |
| Close | 15:00–16:00 | Volume picks up; marking strategy activates |

The **marking the close** manipulation only activates in the Close session. For all other modes, manipulation is active throughout the open trading hours.

---

## Skipping through time

You don't have to watch the whole day in sequence. The **skip bar** (below the manipulation selector) has two types of controls:

**Session presets** jump to specific times instantly:
- **06:00 Pre** — start of day
- **09:39 Open** — just after the market opens
- **12:00 Mid** — mid-session
- **15:00 Close** — start of the close session
- **15:55 Mark** — deep into the settlement window

**Relative buttons** move from wherever you are now:
- **◀◀ −1h** / **◀ −30m** — rewind
- **+30m ▶** / **+1h ▶▶** — skip forward

**The scrubber** — the gradient bar at the right of the skip row — lets you drag to any point. Hover over it to preview the time before clicking.

Skipping to a future time fast-forwards the simulation silently (the agents still make all their decisions, P&L still accumulates, you just don't watch it in real time). Skipping backwards rewinds to a clean state and fast-forwards to the new target.

---

## Trying each manipulation mode

### ⚡ Spoofing — start here

**What to watch:** The event log (bottom panel) will show red entries like:
> *SPOOF: 312-lot BID order placed at 5891.75 by agent MN1. Real ASK order placed at 5892.50.*

A few seconds later:
> *SPOOF CANCEL: BID order at 5891.75 cancelled — order never intended to fill.*

In the order book, look for the **●** markers on the bid side — those are the fake orders. They'll disappear when cancelled.

**Switch to Forensics** (tab in the right panel) to see the cancel ratio building. When MN1's cancel rate exceeds 70%, it's in red — that's the primary spoofing detection signal.

---

### 📚 Layering

Similar to spoofing but the fake orders appear at multiple price levels simultaneously, creating a visible "wall". Watch the order book — you'll see several bid or ask levels all get the ● marker at the same time, then all disappear together.

---

### 🔔 Marking the Close

**First:** Use the **15:00 Close** jump button. Nothing will happen immediately.

**Watch the event log** for purple entries:
> *MARK CLOSE: MN1 placing bid support ahead of settlement window.*

Then jump to **15:55 Mark** to see the aggressive phase:
> *MARK CLOSE [SETTLEMENT WINDOW]: MN1 lifts 87 lots at 5894.25 — +2 ticks.*

**Switch to Forensics** and look at the Close/Mid volume rate ratio. Once it exceeds 1.5×, it turns amber. Above 2× it turns red. This ratio — comparing how much faster trading is happening now versus the mid-session — is the primary forensic signal used by regulators.

Also look at the price chart: you should see the price trend upward in the final minutes without any corresponding news event.

---

### 🏃 Front-Running

**Watch the event log carefully.** The key sequence is:

1. A **pink INFO LEAK** entry appears: *"LARGE CLIENT ORDER: Blackrock (FD2) will execute..."* — this is private information MN1 has received
2. Within 1–2 seconds: *"FRONT-RUN: MN1 takes [direction] position..."*
3. A few seconds later: *"CLIENT ORDER EXEC: Blackrock..."* — the large fund order hits the market
4. Shortly after: *"FRONT-RUN CLOSE: MN1 closes position..."*

The pink entries are what make this scenario educational: the information leak is visible to you as the observer, but it represents a private channel that no surveillance system monitoring only the market data feed can see. This is why front-running prosecution always requires communications evidence.

---

### 🔄 Wash Trading

This one is subtle by design. Watch the time & sales tape — some prints will have a **WASH** tag in pink. These look identical to normal trades. The forensics panel shows the wash volume growing alongside total volume, inflating the reported figures.

The key insight: switch to the Forensics tab and look at "True Volume" vs "Reported Volume". The difference is what wash trading adds — and it's invisible to anyone without cross-account ownership data.

---

### 🔍 Insider Trading

**Skip to 12:00 Mid** to make sure you're past the 30-minute trigger point, then wait and watch the event log.

When the private information event fires, a **pink INFO LEAK** entry will appear describing what IN1 knows. Nothing changes in the price or tape at this moment — IN1 starts quietly accumulating a passive position.

Watch IN1's card in the agent panel: its position (Pos) will grow over the next few minutes. The P&L stays modest because the position is held at entry price.

Then the public announcement fires as a **cyan FLOW** entry. The price moves. IN1 exits. You'll see a large P&L gain crystallise instantly on IN1's card.

**The forensic lesson:** go back through the event log. The time gap between the pink INFO LEAK and the cyan public announcement — during which IN1 was accumulating — is the information advantage window. That accumulation is visible in retrospect but looks like patient directional trading in real time.

---

## Understanding agent cards in depth

Click the **header bar** of any agent card to open a detail panel. This shows:

- **Live stats strip** at the top (P&L, position, fills, bid/ask exposure, capital remaining) — updates every 1.5 seconds
- **Tags** — colour-coded summary of the agent's key properties
- **Overview** — what this agent is and what it's trying to do
- **Live Strategy State** — the actual current values of the agent's internal decision parameters. For example, for a retail agent you can see their current directional signal, whether it's bullish or bearish, and their remaining position limit
- **Decision Rules** — the full table of rules the agent follows
- **Edge & Vulnerability** — what the agent profits from and which manipulation modes hurt it
- **Reference** — the academic paper the agent's design is based on

---

## Forensics tab — what you're looking at

Switch to the Forensics tab (in the right-side panel) while a manipulation simulation is running.

The content changes based on the active mode:

**Cancel Ratio table** (always shown): every agent ranked by how often they cancel orders. Manipulation agents will have very high cancel rates (70–99%); market makers will be absent from this table (their constant quote refreshing is excluded as it's not meaningful cancellation).

**Price Impact table** (always shown): recent fills that moved the price, with how many ticks they moved it and which agent type was responsible.

**Agent Fill Summary** (always shown): who has traded how much, with current resting exposure.

**Mode-specific sections** appear above these and include the key metrics for that manipulation type — the spoofing cancel rate, the marking close/mid volume ratio, the wash true vs reported volume comparison, and so on.

**📖 Why these metrics?** — click this button in the tab bar to open the research modal. This explains the academic basis for each metric: who studied it, what paper it comes from, what the threshold values mean, and what the regulatory implications are.

---

## Downloading data

Three download buttons are available:

**⬇ T&S CSV** — in the right panel header. Downloads every fill printed to the tape since the simulation started: timestamp, sim-second, price, quantity, buyer agent, seller agent, tag (MANIP/FUND/MM/WASH etc.), session, and manipulation mode.

**⬇ SIM CSV** — also in the right panel header. Downloads the full price bar history (every 10 sim-seconds) plus two appended summary blocks: an agent-by-agent table and a session volume breakdown.

**⬇ EVENT LOG CSV** — in the event log panel header. Downloads every entry from the event log: time, type (manip/info/order/close/general), session, mode, and the full text of the event.

These files are useful for:

- Running your own spoofing detection algorithm on the T&S data
- Comparing volume rates across sessions in the SIM CSV
- Building a timeline of information flow for the insider trading scenario from the EVENT LOG CSV

---

## Common questions

**The simulation isn't starting / the order book is empty.**
Click **▶ START** in the top-right of the header bar. The order book only populates when the simulation is running.

**I can't see any manipulation activity.**
Check that you've selected a manipulation mode (default is "No Manipulation"). The manipulation agents are inactive in the default mode. Also check that the simulation has reached the Open session (09:30+) — most manipulation is inactive in pre-market.

**The retail agents keep going bankrupt very quickly.**
This reflects the mathematical reality of retail futures trading. ES futures require $11,500 initial margin per contract, and a 10-tick move against a position costs $125 on a single contract. A $15k account with 1 contract and a 5-tick stop-loss can absorb approximately 24 losing trades before the account is significantly impaired. The simulation uses a realistic capital model — most retail accounts don't survive long in a market with active manipulation.

**Marking the close doesn't seem to do anything.**
This mode only activates in the CLOSE session (15:00–16:00 CT). Use the **15:00 Close** or **15:55 Mark** jump buttons to skip to the relevant time. The forensics comparison also requires data from earlier in the day (mid-session baseline) to be meaningful — the Close/Mid ratio shows 0.0 if there's no mid-session data yet.

**The forensics shows 0% cancel rate for the manipulator.**
This usually means the simulation just started and MN1 hasn't had a full spoof cycle yet (the cycle fires every 120 sim-steps). Wait a minute at 3× speed, or use 8× to advance faster. The cancel rate will populate once at least one complete place-and-cancel cycle has occurred.

**Can I run multiple manipulation modes at once?**
No — the selector is single-choice. Each mode has a distinct agent roster and forensic profile; combining them would make the forensic analysis ambiguous. Run them sequentially and compare the resulting CSV exports if you want to contrast patterns.

---

## Suggested exercises

These exercises work well as standalone exploration or as structured assignments following a lecture.

**Exercise 1 — Spoofing detection (30 minutes)**  
Run the Spoofing mode to the mid-session (use the 12:00 Mid jump). Download the T&S CSV. In a spreadsheet, identify all prints tagged "MANIP" and calculate: (a) the average time between a MANIP fill and the preceding large order placement, (b) the average order size of MANIP fills vs all other fills, (c) the cancel rate you can infer from the event log CSV. Compare your findings to the forensics panel.

**Exercise 2 — Marking the close baseline comparison (20 minutes)**  
Run No Manipulation mode to 16:00. Download the SIM CSV. Note the mid-session volume rate (vol/min from the session breakdown). Restart with Marking the Close mode and jump to 15:00. Watch the close session volume rate in the forensics panel. How does the Close/Mid ratio compare to the 1.5× threshold? At what point in the close session does MN1's share of volume become detectable?

**Exercise 3 — Retail capital survival (15 minutes)**  
Run No Manipulation mode at 20× speed for a full day. Note how many retail agents go bankrupt and in what order. Restart with Spoofing mode and repeat. Does manipulation affect retail agent survival rates? Why might that be, given that the spoof orders are never intended to fill?

**Exercise 4 — Insider vs public information timeline (25 minutes)**  
Run Insider Trading mode. Download the Event Log CSV. Find the row where the INFO LEAK fires. Find the row where the public announcement fires. Calculate the time delta. In the T&S CSV, identify all IN1 fills that occur in this window. What is the total position size accumulated? Calculate the approximate P&L from the subsequent price move.

---

*LOB-SIM is a simulation for educational purposes. All agents, prices, and events are synthetic. The simulation does not connect to any live market data.*
