# 🏦 Finance RAG Agent

A small, self-contained **Retrieval-Augmented Generation (RAG) agent** for finance and risk questions, built with Claude's tool-calling API and real semantic search — no LangChain, no LlamaIndex, no vector DB required to get started.

Given a question, the agent decides for itself — turn by turn — whether it needs to **search** an internal knowledge base, **calculate** something, or **fetch a live stock price**, then reasons over whatever comes back to produce a grounded final answer.

## Why this project

Most beginner "RAG" demos fake retrieval with keyword/dictionary lookup and hide the interesting logic inside a framework. This project is deliberately small and readable end-to-end:

- **Real retrieval** — sentence-transformer embeddings (`all-MiniLM-L6-v2`) + cosine similarity, not string matching.
- **A real agent loop** — Claude's tool-calling API decides which tool (if any) to invoke per question, written out explicitly so the request → tool call → tool result → next request cycle is easy to follow.
- **Easy to extend** — swap in your own documents, add a new tool, or plug in a real vector database, without touching the core loop.

## Demo

```
🙋 What is model risk management and how does it relate to AI validation?
🔧 tool call: knowledge_base_search({'query': 'model risk management AI validation'})
🤖 Model risk management (MRM) is the discipline of identifying, measuring, and
   controlling the risk that a model's errors or misuse lead to adverse outcomes...

🙋 What's the current price of AAPL stock?
🔧 tool call: stock_price({'ticker': 'AAPL'})
🤖 AAPL's last closing price was $XXX.XX.

🙋 If I invest $10000 at 5% annual interest compounded yearly for 3 years...
🔧 tool call: calculator({'expression': '(1.05)**3 * 10000'})
🤖 After 3 years, your $10,000 would grow to approximately $11,576.25.
```

## Architecture

```
                     ┌─────────────────────┐
                     │   User question      │
                     └──────────┬───────────┘
                                ▼
                     ┌─────────────────────┐
                     │   Claude (agent)     │◄──────────────┐
                     │  decides which tool,  │               │
                     │  if any, to call      │               │
                     └──────────┬───────────┘               │
                                ▼                            │
              ┌─────────────────┼─────────────────┐          │
              ▼                 ▼                 ▼          │
   ┌───────────────┐  ┌────────────────┐  ┌───────────────┐  │
   │ knowledge_base │  │   calculator   │  │  stock_price   │  │
   │    _search     │  │  (safe eval)   │  │  (yfinance)    │  │
   │ (embeddings +  │  └────────────────┘  └───────────────┘  │
   │ cosine sim.)   │                                          │
   └───────┬────────┘                                          │
           │                    tool result fed back            │
           └──────────────────────────────────────────────────┘
                                ▼
                     ┌─────────────────────┐
                     │   Final answer       │
                     └─────────────────────┘
```

## Tech stack

| Piece | Tool |
|---|---|
| LLM + agent orchestration | [Anthropic API](https://docs.anthropic.com) (Claude, tool use) |
| Embeddings | [`sentence-transformers`](https://www.sbert.net/) (`all-MiniLM-L6-v2`) |
| Vector search | NumPy cosine similarity (swap for FAISS/Chroma/pgvector at scale) |
| Live market data | [`yfinance`](https://github.com/ranaroussi/yfinance) |
| Safe math | Python `ast` module (whitelisted operators — no `eval()`) |

## Getting started

### 1. Clone and open the notebook

```bash
git clone https://github.com/<your-username>/finance-rag-agent.git
cd finance-rag-agent
jupyter notebook finance_rag_agent.ipynb
```

Or open it directly in **Google Colab** — no local setup required.

### 2. Set your Anthropic API key

Get a key from the [Anthropic Console](https://console.anthropic.com/), then either:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

or just run the notebook — it will prompt for the key securely via `getpass` if the environment variable isn't set. **Never commit your API key to git.**

### 3. Run the cells top to bottom

The first run will download the `all-MiniLM-L6-v2` embedding model (~80MB) and install dependencies. After that, everything runs locally except the calls to Claude and `yfinance`.

## Project structure

```
finance-rag-agent/
├── finance_rag_agent.ipynb   # the whole project — one notebook, six sections
├── requirements.txt          # pinned-ish dependency list for local (non-Colab) use
└── README.md                 # this file
```

## Extending this project

- **Bring your own documents** — replace the hardcoded `KNOWLEDGE_BASE` dict with chunked PDFs (10-Ks, policy manuals, research notes). The retrieval function doesn't need to change.
- **Scale the vector store** — swap the in-memory NumPy array for FAISS, Chroma, or pgvector once the corpus grows past a few hundred chunks.
- **Add more tools** — anything that's a Python function with a JSON schema can be added to `TOOLS`/`TOOL_FUNCTIONS` (e.g. SEC EDGAR filings, FRED economic series, portfolio analytics).
- **Evaluate it** — log retrieved-chunk relevance and tool-choice accuracy against a small labeled test set.
- **Ship a UI** — wrap `run_agent()` in a small Streamlit or Gradio app for an interactive demo.

## Notes & limitations

- The knowledge base is a small in-memory demo corpus (5 entries) meant to show the retrieval mechanism clearly, not to be comprehensive.
- `stock_price` depends on `yfinance` pulling from Yahoo Finance, which is a free/unofficial data source and can occasionally rate-limit or return stale data.
- This is a portfolio/learning project, not investment advice, and the outputs shouldn't be relied on for real financial decisions.

## License

MIT — feel free to fork, adapt, and use this as a starting point for your own projects.
