# 🔬 ProductIQ — AI Product Research Agent

> **Instant deep research on any product** — prices, specs, reviews & where to buy — powered by dual AI search engines and Groq LLaMA 3.3.

![Python](https://img.shields.io/badge/Python-3.9+-blue?style=flat-square&logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-1.32+-red?style=flat-square&logo=streamlit)
![LangChain](https://img.shields.io/badge/LangChain-0.2+-green?style=flat-square)
![Groq](https://img.shields.io/badge/Groq-LLaMA_3.3_70B-orange?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-purple?style=flat-square)

---

## 📸 What It Does

You type a product name. The agent automatically:

1. Searches **DuckDuckGo** for specs, overview, and manufacturer details
2. Searches **Tavily** for deep web results and pricing pages *(optional)*
3. Scrapes **price comparison data** across Amazon, Walmart, Best Buy, eBay, Flipkart
4. Collects **user reviews, expert ratings, pros & cons**
5. Uses **Groq LLaMA 3.3 70B** to synthesize everything into a structured report

**Output report includes:**
- 🏷️ Product Overview
- 💰 Pricing & Where to Buy (with platform links)
- ⚙️ Technical Specifications
- ⭐ Reviews & Ratings
- ✅ Pros & ❌ Cons
- 🛒 All Available Platforms
- 🏆 Final AI Verdict

---

## 🚀 Quick Start

### Step 1 — Clone or Download

```bash
git clone https://github.com/yourname/productiq.git
cd productiq
```

Or just download `product_research_agent.py` and `requirements.txt` into a folder.

---

### Step 2 — Install Dependencies

```bash
pip install -r requirements.txt
```

**What gets installed:**

| Package | Version | Purpose |
|---|---|---|
| `streamlit` | ≥1.32.0 | Web UI framework |
| `langchain-groq` | ≥0.1.3 | Groq LLM integration |
| `langchain-core` | ≥0.2.0 | LangChain base |
| `langchain-community` | ≥0.2.0 | Tavily tool support |
| `duckduckgo-search` | ≥6.1.0 | Free web search |
| `tavily-python` | ≥0.3.3 | Deep search API |
| `requests` | ≥2.31.0 | HTTP calls |

---

### Step 3 — Run the App

```bash
streamlit run product_research_agent.py
```

Then open your browser at **http://localhost:8501**

---

## 🔑 API Keys

ProductIQ needs **1 required** and **1 optional** API key.

### Required — Groq API Key

| Detail | Info |
|---|---|
| **Where to get it** | [console.groq.com](https://console.groq.com) |
| **Cost** | 100% Free |
| **Key format** | Starts with `gsk_` |
| **Credit card?** | Not required |

Steps:
1. Go to [console.groq.com](https://console.groq.com)
2. Sign up with Google or email
3. Click **API Keys** → **Create API Key**
4. Copy the key starting with `gsk_`

---

### Optional — Tavily API Key

| Detail | Info |
|---|---|
| **Where to get it** | [app.tavily.com](https://app.tavily.com) |
| **Cost** | Free tier: 1,000 searches/month |
| **Key format** | Starts with `tvly-` |
| **Why use it?** | Adds deeper, more accurate web results |

Steps:
1. Go to [app.tavily.com](https://app.tavily.com)
2. Sign up for free
3. Copy your API key starting with `tvly-`

---

### How to Enter Keys

**Option A — Enter in the App UI (Recommended)**

When you first launch the app, a setup screen appears asking for your keys. Enter them there — no config files needed.

```
Launch app → Setup Screen → Enter gsk_ key → Enter tvly- key (optional) → Click Launch
```

**Option B — Environment Variables (Advanced)**

Set keys before running so the setup screen is skipped automatically:

```bash
# Mac / Linux
export GROQ_API_KEY="gsk_your_key_here"
export TAVILY_API_KEY="tvly_your_key_here"
streamlit run product_research_agent.py
```

```cmd
:: Windows Command Prompt
set GROQ_API_KEY=gsk_your_key_here
set TAVILY_API_KEY=tvly_your_key_here
streamlit run product_research_agent.py
```

```powershell
# Windows PowerShell
$env:GROQ_API_KEY = "gsk_your_key_here"
$env:TAVILY_API_KEY = "tvly_your_key_here"
streamlit run product_research_agent.py
```

**Option C — .env File**

Create a `.env` file in the same folder as the script:

```env
GROQ_API_KEY=gsk_your_key_here
TAVILY_API_KEY=tvly_your_key_here
```

Then add this at the top of `product_research_agent.py`:

```python
from dotenv import load_dotenv
load_dotenv()
```

And install python-dotenv:

```bash
pip install python-dotenv
```

> ⚠️ **Never commit your `.env` file to GitHub.** Add it to `.gitignore`.

---

## 🏗️ How It Works — Architecture

```
User Input
    │
    ▼
┌─────────────────────────────────────────────────┐
│              Streamlit UI Layer                 │
│   Setup Screen → Main App → Results Display    │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│           ProductResearchAgent                  │
│                                                 │
│  AgentState = { messages: [list + operator.add]}│
│                                                 │
│  ┌──────────┐    ┌─────────────────────────┐   │
│  │ call_llm │◄───│   take_action (tools)   │   │
│  │  (Groq)  │    │                         │   │
│  └────┬─────┘    │  • search_duckduckgo    │   │
│       │          │  • search_tavily        │   │
│  has tool_calls? │  • search_prices        │   │
│  YES ─┼─────────►│  • search_reviews       │   │
│  NO   │          └─────────────────────────┘   │
│       ▼                                         │
│  Final Answer                                   │
└─────────────────────────────────────────────────┘
                     │
                     ▼
            Markdown Report Rendered
            + Download as .md file
```

### Agent Loop Explained

The agent runs in a loop up to `max_iterations` times:

```
1. LLM receives: system prompt + full message history
2. LLM responds with: tool calls OR final answer
3. If tool calls → execute all tools → append results to state → go to step 1
4. If no tool calls → render the final answer → stop
```

This means the agent can call **multiple tools in sequence** and **combine all results** before writing the final report.

---

## 🔧 The 4 Search Tools

| Tool | Engine | What it searches for | API Key needed? |
|---|---|---|---|
| `search_duckduckgo` | DuckDuckGo | General specs, overview, manufacturer info | ❌ Free |
| `search_tavily` | Tavily | Deep web pages, detailed product data | ✅ Optional |
| `search_prices` | DuckDuckGo | Amazon, Walmart, eBay, Best Buy, Flipkart prices | ❌ Free |
| `search_reviews` | DuckDuckGo | Expert reviews, user ratings, pros/cons | ❌ Free |

---

## ⚙️ Settings & Configuration

All configurable from the sidebar inside the app:

| Setting | Options | Default | Description |
|---|---|---|---|
| **Groq Model** | 4 options | `llama-3.3-70b-versatile` | LLM used for reasoning |
| **Use Tavily** | Toggle | On if key set | Enable/disable Tavily search |
| **Max Iterations** | 3–10 | 6 | How many search cycles the agent runs |

### Available Groq Models

| Model | Speed | Quality | Best for |
|---|---|---|---|
| `llama-3.3-70b-versatile` | Medium | ⭐⭐⭐⭐⭐ | Best overall — recommended |
| `llama3-70b-8192` | Medium | ⭐⭐⭐⭐ | Good alternative |
| `mixtral-8x7b-32768` | Fast | ⭐⭐⭐⭐ | Great tool use |
| `llama-3.1-8b-instant` | Very fast | ⭐⭐⭐ | Quick lightweight searches |

---

## 📁 Project Structure

```
productiq/
│
├── product_research_agent.py   # Main app — all code in one file
├── requirements.txt            # Python dependencies
├── README.md                   # This file
└── .env                        # Your API keys (never commit this!)
```

---

## 💡 Usage Examples

Try these queries in the app:

```
Samsung Galaxy S24 Ultra
Apple MacBook Pro M3 Max
Sony WH-1000XM5 headphones
Dyson V15 Detect vacuum cleaner
NVIDIA RTX 4090 GPU
iPhone 16 Pro Max
LG OLED C3 55 inch TV
DJI Mini 4 Pro drone
Kindle Paperwhite 2024
Bosch Series 8 washing machine
```

---

## 🛒 Platforms Detected

The agent looks for pricing and availability on:

- 🟠 **Amazon** (global)
- 🔵 **Walmart** (US)
- 🔴 **eBay** (global)
- 🔵 **Best Buy** (US/Canada)
- 🔵 **Flipkart** (India)
- 🟠 **Newegg** (tech products)
- Any other retailer found in search results

---

## ❓ Troubleshooting

### App won't start
```bash
# Make sure all packages are installed
pip install -r requirements.txt

# Check Python version (need 3.9+)
python --version
```

### `gsk_` key not working
- Make sure you copied the full key from [console.groq.com](https://console.groq.com)
- Keys expire if unused — generate a new one
- Check you're on the free tier (not exceeded limits)

### DuckDuckGo search errors
```bash
# Upgrade duckduckgo-search to latest
pip install --upgrade duckduckgo-search
```
DuckDuckGo occasionally rate-limits. Wait 60 seconds and try again.

### Tavily returns no results
- Verify key starts with `tvly-` not `gsk_`
- Check your monthly limit at [app.tavily.com](https://app.tavily.com)
- The app works fine without Tavily — just disable the toggle

### Agent gives incomplete report
- Increase **Max Search Iterations** to 8–10 in the sidebar
- Try a more specific product name (include brand + model number)
- Switch to `llama-3.3-70b-versatile` model for best results

### Port already in use
```bash
# Run on a different port
streamlit run product_research_agent.py --server.port 8502
```

---

## 🔒 Privacy & Security

- **No data stored** — all results exist only in your browser session
- **Keys never leave your machine** — stored only in `st.session_state` (browser RAM)
- **No database** — closing the browser tab clears everything
- **No telemetry** — the app makes no calls except to Groq, Tavily, and DuckDuckGo

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **UI** | Streamlit 1.32+ |
| **LLM** | Groq API — LLaMA 3.3 70B |
| **Agent Framework** | LangChain Core + LangChain Groq |
| **Search 1** | DuckDuckGo Search (free, no key) |
| **Search 2** | Tavily API (optional, deeper results) |
| **State Management** | LangGraph-style `AgentState` with `operator.add` |
| **Fonts** | Syne + DM Sans (Google Fonts) |
| **Theme** | Custom dark luxury CSS |

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 🙌 Credits

Built with:
- [Streamlit](https://streamlit.io) — UI framework
- [Groq](https://groq.com) — ultra-fast LLM inference
- [LangChain](https://langchain.com) — agent framework
- [Tavily](https://tavily.com) — AI-optimized search API
- [DuckDuckGo](https://duckduckgo.com) — free privacy-first search

---

<div align="center">
Made with ❤️ | ProductIQ v1.0
</div>
