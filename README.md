# 🇵🇰 Pakistan REER Calculator
### A Python-based financial modeling tool for tracking Pakistan's real currency competitiveness

---

## Why I Built This

The project started as a challangen from my manager to find the true value of 

The nominal PKR/USD exchange rate tells you almost nothing on its own. A currency can be losing value against the dollar every single day and *still* be getting more expensive in real terms — if domestic inflation is outpacing everyone else's. To make an informed call on trade competitiveness, pricing strategy, or import/export exposure, you need a number that strips out the noise and tells you: **relative to the currencies of the countries we actually trade with, adjusted for inflation, is the Rupee cheap or expensive right now?**

That number is called the **Real Effective Exchange Rate (REER)** — and rather than relying on a single published monthly figure from the State Bank of Pakistan (SBP), I built a tool that calculates it independently, from live data, so the business can track it in near real-time and understand exactly what's driving it.

> **This project isn't just a script — it's a full data pipeline: web scraping, API integration, financial index construction, and a validated economic model, all built to answer a real business question.**

---

## What It Actually Does

This tool:
1. **Pulls Pakistan's top 36 trading partners** and their official trade weights
2. **Fetches live inflation data (CPI)** for every country from the World Bank's global database
3. **Fetches live and historical foreign exchange rates** for every partner currency
4. **Calculates the Real Effective Exchange Rate (REER)** — a single number that tells you whether the Rupee is over- or under-valued in real, trade-weighted terms
5. **Validates the result** against the State Bank of Pakistan's own published REER index

The final output isn't just a number — it's a reproducible, automatable model that can be re-run any time new data is needed, instead of waiting on SBP's monthly release.

📸 *[Insert a screenshot here of the final DataFrame output / REER result printout]*

---

## The Core Concepts (Explained Simply)

If you're a recruiter or hiring manager skimming this — here's the 60-second version of the economics behind the code.

### CPI — Consumer Price Index
A measure of how expensive a fixed "basket" of everyday goods and services is in a country, tracked over time. If CPI rises, prices are rising — that's inflation.

### RPI — Relative Price Index
This isn't the UK "Retail Price Index" — in this context, RPI means comparing **Pakistan's inflation to a trading partner's inflation**. If Pakistan's CPI has grown faster than, say, Germany's since a shared reference point, Pakistani goods have gotten relatively more expensive compared to German goods — even before touching exchange rates at all.

### NEER — Nominal Effective Exchange Rate
A trade-weighted average of the Rupee's exchange rate against a basket of partner currencies. Useful, but incomplete — it only tells you how the *currency* has moved, not whether goods priced in that currency have actually gotten cheaper or more expensive to buy.

### REER — Real Effective Exchange Rate
**REER = NEER, adjusted for inflation differentials.**

This is the number that actually matters for competitiveness. Two countries can have identical exchange rate movements, but if one has much higher inflation, its real purchasing power and trade competitiveness are completely different. REER captures *both* effects — currency movement **and** relative inflation — in a single trade-weighted index.

**Why REER beats NEER for real decision-making:** NEER can make a currency look stable or even strong, while inflation quietly erodes its real competitiveness underneath. REER is the number central banks, investors, and multinational finance teams actually watch, because it reflects what's *actually* happening to purchasing power and export competitiveness — not just what the exchange rate ticker says.

📸 *[Insert the REER formula image here — CPI_home/CPI_i × FX ratio, raised to trade-weight, product across all partners]*

---

## How It Works (Technical Overview)

**Tech stack:** Python · pandas · NumPy · `yfinance` · World Bank API · `requests` · `curl_cffi`

The pipeline runs in five stages:

| Stage | What Happens | Key Library |
|---|---|---|
| 1. Trade Basket | Load Pakistan's top 36 trading partners + official trade weights | `pandas`, Excel |
| 2. Inflation Data | Pull each country's CPI (rebased to a common year) from the World Bank API | `requests` |
| 3. Exchange Rates | Pull current and historical FX rates for every partner currency | `yfinance` |
| 4. Index Construction | Compute each country's Relative Price Index and Nominal Exchange Rate Index | `pandas`, `numpy` |
| 5. Aggregation | Combine into a single trade-weighted REER using a weighted geometric mean | `numpy` |

The final REER is calculated as a **weighted geometric mean** across all 36 trading partners — each partner's contribution to the index is scaled by how much trade Pakistan actually does with them, so a small trading partner having wild inflation doesn't distort the whole picture, while a dominant partner (like China, at ~32% of trade weight) appropriately drives more of the result.

---

## Validation — Does It Actually Work?

The model was cross-checked against the State Bank of Pakistan's officially published REER index.

**Result: 102.49**, landing squarely within SBP's own published range for the same period (roughly 102–106 across recent months) — a strong signal that the independently-built model is capturing the same underlying economic reality as the central bank's official calculation.

📸 *[Insert a comparison chart here: your REER output vs. SBP's published REER graph over time]*

---

## The Debugging Journey (Because Real Projects Aren't Clean)

This model didn't come out right on the first try — and I think that's worth documenting rather than hiding:

- **First pass**, a naive HTML-scraping approach using fragile character-position slicing broke the moment the source website changed layout — replaced with a proper `pandas.read_html` table-parsing pipeline.
- **CPI values initially looked nonsensical** (a "223% inflation differential" that made no real-world sense) — traced back to a subtle but critical issue: raw CPI index points from different countries are rebased to *different, arbitrary base years* by each country's own statistics agency, making direct comparisons meaningless. Fixed by switching to the World Bank's globally standardized CPI series (2010 = 100 for every country).
- **Yahoo Finance FX data intermittently failed** with cryptic "possibly delisted" errors — diagnosed as Yahoo-side rate-limiting/bot-detection, resolved using a browser-impersonating request session.
- **First REER estimate landed at 86** — a red flag, since it didn't match reality. Root-caused to an asymmetry between the FX leg (correctly rebased to 2010) and the CPI leg (not rebased at all). Fixing the CPI source to match the FX leg's base year brought the model in line with SBP's real published figures.

Each of these was a genuine "why is this number wrong" investigation rather than a guess-and-check fix — which is exactly the kind of debugging discipline this project was meant to demonstrate.

---

## Getting Started

```bash
# Clone the repo
git clone https://github.com/yourusername/pakistan-reer-calculator.git
cd pakistan-reer-calculator

# Install dependencies
pip install pandas numpy requests yfinance curl_cffi openpyxl

# Run the notebook
jupyter notebook REER_Calculator.ipynb
```

**Inputs required:**
- `Trading_Partners.xlsx` — list of trading partners, ISO3 codes, currency codes, and trade weights

**Outputs:**
- `REER_Calculated_Results.csv` — full breakdown per country
- Console output of the final REER value

---

## Limitations & Future Improvements

- World Bank CPI data is **annual**, while SBP's internal calculation uses monthly data — this model is a strong approximation, not a monthly-precision replica.
- Taiwan is excluded from the CPI dataset (not covered by the World Bank, as it isn't a member state) — its small trade weight (~1.3%) is redistributed proportionally across the remaining partners.
- **Next steps:** automate a scheduled monthly run and log historical REER values to build an independent time series; source a monthly CPI feed to close the frequency gap with SBP's methodology.

---

## Why This Project Matters

This wasn't a toy exercise — it was built to answer a real question a manager needed answered, using publicly available data, and it was validated against the actual number a country's central bank publishes. It demonstrates the full lifecycle of a data project: sourcing messy real-world data, catching and fixing conceptual (not just syntax) errors, and validating the output against ground truth before trusting it for a business decision.
