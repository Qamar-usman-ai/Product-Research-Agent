import os
import re
import time
import operator
import warnings
from typing import Annotated, TypedDict, List

import streamlit as st

# ============================================================
#  PAGE CONFIG (MUST BE THE VERY FIRST STREAMLIT CALL)
# ============================================================
st.set_page_config(
    page_title="ProductIQ — AI Research Agent",
    page_icon="🔬",
    layout="wide",
    initial_sidebar_state="expanded"
)

from langchain_groq import ChatGroq
from langchain_core.messages import BaseMessage, HumanMessage, SystemMessage, ToolMessage, AIMessage
from langchain_core.tools import tool
from duckduckgo_search import DDGS

warnings.filterwarnings("ignore", category=DeprecationWarning)

# ============================================================
#  CUSTOM CSS — Dark luxury theme
# ============================================================
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Sans:ital,wght@0,300;0,400;0,500;1,300&display=swap');

html, body, [class*="css"] { font-family: 'DM Sans', sans-serif; }
.stApp { background: #080b14; color: #e8eaf0; }
.block-container { padding-top: 2rem !important; max-width: 1200px; }

[data-testid="stSidebar"] { background: #0d1117 !important; border-right: 1px solid #1e2535; }
[data-testid="stSidebar"] * { color: #c9cdd8 !important; }
[data-testid="stSidebar"] h2, [data-testid="stSidebar"] h3 { font-family: 'Syne', sans-serif !important; color: #ffffff !important; }

.hero { text-align: center; padding: 3rem 1rem 2rem; position: relative; }
.hero-badge {
    display: inline-block;
    background: linear-gradient(135deg, rgba(99,102,241,0.2), rgba(168,85,247,0.2));
    border: 1px solid rgba(99,102,241,0.4);
    border-radius: 20px;
    padding: 4px 16px;
    font-size: 0.72rem;
    text-transform: uppercase;
    color: #a78bfa;
    margin-bottom: 1.2rem;
}
.hero-title {
    font-family: 'Syne', sans-serif;
    font-size: 3.4rem;
    font-weight: 800;
    line-height: 1.1;
    background: linear-gradient(135deg, #ffffff 0%, #a78bfa 50%, #6366f1 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin: 0 0 1rem;
}
.stTextInput > div > div > input {
    background: #111827 !important;
    border: 1.5px solid #1e2535 !important;
    border-radius: 12px !important;
    color: #e8eaf0 !important;
}
.stButton > button {
    background: linear-gradient(135deg, #6366f1, #8b5cf6) !important;
    color: white !important;
    border-radius: 12px !important;
    font-family: 'Syne', sans-serif !important;
    width: 100% !important;
    box-shadow: 0 4px 20px rgba(99,102,241,0.3) !important;
}
.result-card {
    background: #0d1117;
    border: 1px solid #1e2535;
    border-radius: 16px;
    padding: 1.5rem 1.8rem;
    margin-bottom: 1.2rem;
    position: relative;
    overflow: hidden;
}
.result-card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 3px;
    background: linear-gradient(90deg, #6366f1, #8b5cf6, #ec4899);
}
.tool-log {
    background: #070a0f;
    border: 1px solid #1a2030;
    border-radius: 12px;
    padding: 1rem 1.2rem;
    font-family: monospace;
    font-size: 0.8rem;
    color: #4ade80;
}
.fancy-divider { height: 1px; background: linear-gradient(90deg, transparent, #1e2535, transparent); margin: 1.5rem 0; }
</style>
""", unsafe_allow_html=True)

# ============================================================
#  STATE MANAGEMENT
# ============================================================
if "history" not in st.session_state: st.session_state.history = []
if "tool_logs" not in st.session_state: st.session_state.tool_logs = []
if "keys_saved" not in st.session_state: st.session_state.keys_saved = False
if "groq_key" not in st.session_state: st.session_state.groq_key = os.environ.get("GROQ_API_KEY", "")
if "tavily_key" not in st.session_state: st.session_state.tavily_key = os.environ.get("TAVILY_API_KEY", "")

if st.session_state.groq_key and len(st.session_state.groq_key) > 10:
    st.session_state.keys_saved = True

# ============================================================
#  TOOLS
# ============================================================
@tool
def search_duckduckgo(query: str) -> str:
    """Search DuckDuckGo for product information, prices, reviews, and specs."""
    try:
        with DDGS() as ddgs:
            results = list(ddgs.text(query, max_results=6))
        if not results: return "No results found."
        return "\n---\n".join([f"TITLE: {r.get('title')}\nINFO: {r.get('body')}\nURL: {r.get('href')}" for r in results])
    except Exception as e: return f"Error: {e}"

@tool
def search_tavily(query: str) -> str:
    """Search Tavily for deep product research and manufacturer details."""
    try:
        from langchain_community.tools.tavily_search import TavilySearchResults
        key = st.session_state.get("tavily_key")
        if not key: return "Tavily key missing."
        os.environ["TAVILY_API_KEY"] = key
        tavily = TavilySearchResults(max_results=5)
        results = tavily.invoke(query)
        return "\n---\n".join([f"INFO: {r.get('content')[:400]}\nURL: {r.get('url')}" for r in results])
    except Exception as e: return f"Error: {e}"

@tool
def search_prices(product: str) -> str:
    """Search for current prices and where to buy."""
    try:
        with DDGS() as ddgs:
            q = f"buy {product} price USD site:amazon.com OR site:walmart.com OR site:bestbuy.com OR site:ebay.com"
            results = list(ddgs.text(q, max_results=6))
        return "\n---\n".join([f"STORE: {r.get('title')}\nDETAILS: {r.get('body')}\nLINK: {r.get('href')}" for r in results])
    except Exception as e: return f"Error: {e}"

@tool
def search_reviews(product: str) -> str:
    """Search for user reviews and ratings."""
    try:
        with DDGS() as ddgs:
            results = list(ddgs.text(f"{product} review rating pros cons 2025", max_results=6))
        return "\n---\n".join([f"SOURCE: {r.get('title')}\nREVIEW: {r.get('body')}\nURL: {r.get('href')}" for r in results])
    except Exception as e: return f"Error: {e}"

# ============================================================
#  AGENT LOGIC
# ============================================================
SYSTEM_PROMPT = """You are ProductIQ, an elite AI product analyst. Collect data using all tools, then write a report with sections: 
## 🏷️ Product Overview, ## 💰 Pricing & Where to Buy, ## ⚙️ Technical Specifications, ## ⭐ Reviews & Ratings, ## ✅ Pros, ## ❌ Cons, ## 🛒 Available Platforms, ## 🏆 Final Verdict."""

class AgentState(TypedDict):
    messages: Annotated[List[BaseMessage], operator.add]

class ProductResearchAgent:
    def __init__(self, groq_key, model, use_tavily):
        all_tools = [search_duckduckgo, search_prices, search_reviews]
        if use_tavily: all_tools.append(search_tavily)
        self.tools_dict = {t.name: t for t in all_tools}
        self.llm = ChatGroq(model=model, temperature=0.1, groq_api_key=groq_key).bind_tools(all_tools)

    def run(self, query: str, max_iter: int = 6):
        state = {"messages": [HumanMessage(content=query)]}
        for _ in range(max_iter):
            msgs = [SystemMessage(content=SYSTEM_PROMPT)] + state["messages"]
            response = self.llm.invoke(msgs)
            state["messages"].append(response)
            if not response.tool_calls: return response.content
            
            for tc in response.tool_calls:
                tool_obj = self.tools_dict.get(tc["name"])
                obs = tool_obj.invoke(tc["args"]) if tool_obj else "Tool not found"
                st.session_state.tool_logs.append(f"🔧 {tc['name']}({tc['args']})")
                state["messages"].append(ToolMessage(tool_call_id=tc["id"], content=str(obs)))
        return self.llm.invoke([SystemMessage(content=SYSTEM_PROMPT)] + state["messages"]).content

# ============================================================
#  UI RENDERING
# ============================================================
def render_report(report, product_name):
    st.markdown(f"<h2 style='text-align:center;'>{product_name}</h2>", unsafe_allow_html=True)
    parts = re.split(r'\n## ', report)
    for part in parts:
        if not part.strip(): continue
        lines = part.strip().split("\n")
        title = lines[0].strip("#").strip()
        body = "\n".join(lines[1:]).strip()
        st.markdown(f'<div class="result-card"><h3>{title}</h3><p>{body}</p></div>', unsafe_allow_html=True)

# ============================================================
#  MAIN APP
# ============================================================
if not st.session_state.keys_saved:
    st.title("🔑 ProductIQ Setup")
    g_input = st.text_input("Groq API Key", type="password")
    t_input = st.text_input("Tavily API Key (Optional)", type="password")
    if st.button("Launch"):
        if g_input.startswith("gsk_"):
            st.session_state.groq_key = g_input
            st.session_state.tavily_key = t_input
            st.session_state.keys_saved = True
            st.rerun()
        else: st.error("Invalid Groq Key")
    st.stop()

# Sidebar
with st.sidebar:
    st.title("🔬 Settings")
    model_choice = st.selectbox("Model", ["llama-3.3-70b-versatile", "llama-3.1-8b-instant"])
    use_tav = st.toggle("Use Tavily", value=bool(st.session_state.tavily_key))
    if st.button("Reset Session"):
        st.session_state.keys_saved = False
        st.rerun()

# Main UI
st.markdown('<div class="hero"><div class="hero-title">ProductIQ</div></div>', unsafe_allow_html=True)
query = st.text_input("What product are we researching today?", placeholder="e.g. Sony WH-1000XM5")

if st.button("Research →") and query:
    st.session_state.tool_logs = []
    with st.status("Agent working...", expanded=True) as status:
        agent = ProductResearchAgent(st.session_state.groq_key, model_choice, use_tav)
        final_report = agent.run(query)
        status.update(label="Complete!", state="complete")
    
    if st.session_state.tool_logs:
        with st.expander("View Logs"):
            for log in st.session_state.tool_logs: st.code(log)
    
    render_report(final_report, query)
