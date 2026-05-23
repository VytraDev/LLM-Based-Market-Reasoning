# LLM-Based Market Reasoning Engine

A Streamlit web app that uses LLM causal reasoning to assess potential stock price movements based on real-world events and news. Built as a proof of concept and learning tool for event-driven macro analysis.

> Built with Python, Streamlit, Gemini API, NewsAPI, yfinance, and SQLite.

---

## The Approach

Most AI stock prediction tools try to find patterns in historical price data. This one does not.

Pattern-based prediction has a well-documented ceiling: markets are filled with professionals and algorithms all reading the same data, so any discoverable edge gets arbitraged away. Price patterns, by definition, are already priced in.

This tool takes a different approach: **event-driven causal reasoning**. Instead of asking "what has this stock done before?", it asks "given this real-world event, which companies and sectors are causally affected, and in which direction?" The reasoning is transparent and traceable, not a black box.

The canonical example: a shipping disruption in the Red Sea affects tanker companies (DHT Holdings, Frontline, Nordic American Tankers) through a specific causal chain — reduced supply, higher day rates, increased revenue. An LLM can trace that chain. A pattern-matching model cannot.

---

## Stack

| Component | Technology |
|---|---|
| UI / dashboard | Streamlit |
| LLM reasoning | Gemini API (`gemini-2.0-flash`, direct REST) |
| Alternative LLM | DeepSeek API (cost-effective swap) |
| News fetching | NewsAPI |
| Price data | yfinance |
| Storage | SQLite |
| Scheduling | APScheduler |
| Visualisation | Plotly |

---

## Features

- **Event input:** paste any real-world event or news headline as plain text
- **LLM reasoning pipeline:** the model identifies affected sectors, specific tickers, directional bias (bullish/bearish), confidence level, and a plain-language causal explanation
- **Price tracking:** fetches current and historical price data for flagged tickers via yfinance
- **Exit signal monitor:** tracks flagged positions and flags when thesis conditions change
- **SQLite logging:** all analyses, price snapshots, and exit signals are stored locally for review
- **Streamlit dashboard:** view active theses, historical analyses, and price movement since flag date
- **Plotly charts:** price movement overlaid with thesis entry points

---

## Setup

### Prerequisites

- Python 3.9+
- A Gemini API key ([aistudio.google.com](https://aistudio.google.com)) — free tier works
- A NewsAPI key ([newsapi.org](https://newsapi.org)) — free tier works for development

### Installation

```bash
git clone https://github.com/VytraDev/LLM-Based-Market-Reasoning.git
cd LLM-Based-Market-Reasoning
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file in the root:

```env
GEMINI_API_KEY=your_gemini_api_key
NEWS_API_KEY=your_newsapi_key
# Optional — DeepSeek as alternative LLM
DEEPSEEK_API_KEY=your_deepseek_api_key
```

### Run

```bash
streamlit run app.py
```

---

## How It Works

1. **News fetcher** pulls recent headlines from NewsAPI using OR-joined keyword queries (one API call per run, not per keyword — avoids daily quota issues)
2. **Gemini reasoner** receives each headline and returns structured JSON: affected tickers, directional bias, confidence, causal chain explanation
3. **Price tracker** fetches current price data for flagged tickers via yfinance and stores snapshots in SQLite
4. **Exit signal monitor** re-evaluates open theses on each run and flags positions where the original thesis conditions no longer hold
5. **Streamlit dashboard** surfaces all of this with Plotly charts showing price movement since each thesis was logged

---

## Honest Assessment

This is a proof of concept and learning tool, not a trading system.

With the free NewsAPI tier, the realistic directional accuracy ceiling is **55–65%**. The bottleneck is news latency: by the time a free-tier API surfaces a headline, professional traders have already acted on it. The causal reasoning is sound; the data pipeline is slow.

What the tool does well:
- Teaching event-to-sector causal chains through real, live examples
- Generating a structured record of thesis reasoning that can be reviewed and calibrated over time
- Demonstrating that LLM reasoning is a defensible approach to macro analysis, even if the current data pipeline limits its practical edge

What would move it closer to production:
- A premium news source with lower latency (Bloomberg, Benzinga, Polygon.io)
- A backtesting layer against historical news archives and price data
- Sector-level filtering based on observed model performance over time

---

## Project Structure

```
LLM-Based-Market-Reasoning/
├── app.py                  # Streamlit dashboard entry point
├── config.py               # API keys, constants, model selection
├── database.py             # SQLite schema and query helpers
├── news_fetcher.py         # NewsAPI integration (OR-joined queries)
├── reasoner.py             # Gemini API reasoning pipeline
├── price_tracker.py        # yfinance price snapshot logic
├── exit_monitor.py         # Thesis exit signal evaluation
├── requirements.txt
└── .env.example
```

---

## Notes

- The Gemini integration uses direct HTTP REST calls, not the `google-generativeai` SDK (migrated due to SDK deprecation)
- DeepSeek (`deepseek-chat`) is supported as a drop-in alternative — roughly the same reasoning quality at significantly lower API cost
- All analysis outputs are stored in SQLite; no external database required
