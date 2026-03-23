# 🔍 Local Deep Researcher

> A fully local, privacy-first web research assistant powered by any LLM hosted via [Ollama](https://ollama.com/search) or [LMStudio](https://lmstudio.ai/).

Provide a research topic and the assistant will autonomously generate search queries, gather and summarize web results, identify knowledge gaps, and iterate — delivering a final, citation-rich markdown report without sending your data to any external AI provider.

---

## ✨ Features

- **Fully local** — all inference runs on your machine via Ollama or LMStudio
- **Iterative research loop** — automatically reflects on summaries and refines queries across configurable cycles
- **Multiple search backends** — DuckDuckGo (default, no API key), Tavily, Perplexity, and SearXNG
- **Structured output** — final markdown report with cited sources
- **LangGraph Studio UI** — visualize and control the research graph in real time
- **Tool calling support** — compatible with models that support function/tool calling

---

## 🔥 Changelog

| Date | Update |
|------|--------|
| Aug 6, 2025 | Added tool calling support and [gpt-oss](https://openai.com/index/introducing-gpt-oss/) compatibility |

> ⚠️ **Note (Aug 6, 2025):** The `gpt-oss` models do not support JSON mode in Ollama. Enable `use_tool_calling` in your configuration to use tool calling instead of JSON mode.

---

## 🚀 Quickstart

### 1. Clone the Repository

```shell
git clone https://github.com/langchain-ai/local-deep-researcher.git
cd local-deep-researcher
```

### 2. Configure Environment Variables

```shell
cp .env.example .env
```

Edit `.env` to set your preferred LLM provider, search tool, and other options. Values set here take precedence over defaults defined in `configuration.py`. They are loaded automatically at runtime via `python-dotenv`.

---

## 🤖 LLM Provider Setup

### Option A — Ollama

1. Download the Ollama app for [Mac](https://ollama.com/download) or your platform.

2. Pull a model:
```shell
ollama pull deepseek-r1:8b
```

3. Configure `.env`:
```shell
LLM_PROVIDER=ollama
OLLAMA_BASE_URL=http://localhost:11434   # default
LOCAL_LLM=deepseek-r1:8b                # defaults to llama3.2 if unset
```

---

### Option B — LMStudio

1. Download and install [LMStudio](https://lmstudio.ai/).

2. Load your preferred model (e.g., `qwen_qwq-32b`) and start the local server under the **Local Server** tab (default port: `1234`).

3. Configure `.env`:
```shell
LLM_PROVIDER=lmstudio
LOCAL_LLM=qwen_qwq-32b                  # use exact model name shown in LMStudio
LMSTUDIO_BASE_URL=http://localhost:1234/v1
```

---

## 🔎 Search Backend Configuration

DuckDuckGo is used by default and requires no API key. To use an alternative backend, add the corresponding key to `.env`:

```shell
SEARCH_API=duckduckgo          # options: duckduckgo | tavily | perplexity | searxng
TAVILY_API_KEY=your_key
PERPLEXITY_API_KEY=your_key
MAX_WEB_RESEARCH_LOOPS=3       # number of research iterations (default: 3)
FETCH_FULL_PAGE=false          # fetch full page content with DuckDuckGo (default: false)
```

---

## 🖥️ Running with LangGraph Studio

### macOS

```bash
# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate

# Launch the LangGraph server
curl -LsSf https://astral.sh/uv/install.sh | sh
uvx --refresh --from "langgraph-cli[inmem]" --with-editable . --python 3.11 langgraph dev
```

### Windows

```powershell
# Create and activate a virtual environment
python -m venv .venv
.venv\Scripts\Activate.ps1

# Install dependencies and start the server
pip install -e .
pip install -U "langgraph-cli[inmem]"
langgraph dev
```

---

### Using the Studio UI

Once the server starts, the following endpoints will be available:

```
API:    http://127.0.0.1:2024
Docs:   http://127.0.0.1:2024/docs
Studio: https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024
```

Open the **LangGraph Studio Web UI** link in your browser. Use the **Configuration** tab to adjust assistant settings directly in the UI.

**Configuration priority order:**
```
1. Environment variables         ← highest priority
2. LangGraph Studio UI settings
3. Default values in Configuration class
```

Enter a research topic and watch the agent reason through its search iterations in real time.

---

## ⚙️ How It Works

Local Deep Researcher is inspired by [IterDRAG](https://arxiv.org/html/2410.04343v1), an approach that decomposes queries into sub-queries, retrieves documents iteratively, and builds on intermediate answers.

The research loop proceeds as follows:

1. **Query generation** — the LLM generates an initial web search query from your topic
2. **Web search** — the configured search engine retrieves relevant sources
3. **Summarization** — the LLM summarizes findings relative to the research topic
4. **Reflection** — the LLM identifies knowledge gaps in the current summary
5. **Refinement** — a new query targets those gaps and the loop repeats
6. **Report generation** — after `N` iterations, a final markdown report is produced with all citations

---

## 📤 Output

The final output is a markdown document containing a structured research summary with inline citations. All sources collected during the session are stored in the graph state, which can be inspected directly in LangGraph Studio.

---

## ♻️ Model Compatibility

Some models may struggle to produce structured JSON output required by certain steps. The assistant includes fallback mechanisms to handle this gracefully. Models known to have difficulty with JSON output include DeepSeek R1 (7B) and DeepSeek R1 (1.5B).

---

## 🌐 Browser Compatibility

| Browser | Status |
|---------|--------|
| Firefox | ✅ Recommended |
| Chrome / Edge | ✅ Supported |
| Safari | ⚠️ May encounter mixed-content warnings (HTTPS/HTTP) |

If you experience issues, try disabling ad-blocking extensions or checking the browser console for errors.

---

## 🐳 Docker Deployment

The included `Dockerfile` runs LangGraph Studio with Local Deep Researcher as a service. Ollama must be run separately.

```bash
# Build the image
docker build -t local-deep-researcher .

# Run the container
docker run --rm -it -p 2024:2024 \
  -e SEARCH_API=tavily \
  -e TAVILY_API_KEY=tvly-YOUR_KEY_HERE \
  -e LLM_PROVIDER=ollama \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434/ \
  -e LOCAL_LLM=llama3.2 \
  local-deep-researcher
```

> **Note:** The container will not open a browser automatically. Navigate manually to:
> [`https://smith.langchain.com/studio/thread?baseUrl=http://127.0.0.1:2024`](https://smith.langchain.com/studio/thread?baseUrl=http://127.0.0.1:2024)

---

## 🔗 Additional Resources

- **TypeScript port** (without Perplexity): [PacoVK/ollama-deep-researcher-ts](https://github.com/PacoVK/ollama-deep-researcher-ts)
- **Deployment options**: [LangGraph Deployment Docs](https://langchain-ai.github.io/langgraph/concepts/#deployment-options) — see Module 6 of [LangChain Academy](https://github.com/langchain-ai/langchain-academy/tree/main/module-6)
- **Research paper**: [IterDRAG (arXiv)](https://arxiv.org/html/2410.04343v1)

---

## 📄 License

See [LICENSE](./LICENSE) for details.
