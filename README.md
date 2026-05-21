# HighHEM

> **Minimal FastAPI Implementation Demonstrating Cost Savings & Quality Improvements with Semantic Highlighting and HHEM**

[![MeshAPI Powered](https://img.shields.io/badge/Powered%20By-MeshAPI-blueviolet?style=for-the-badge&logo=openai)](https://meshapi.ai)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Qdrant](https://img.shields.io/badge/Vector%20Database-Qdrant-red?style=for-the-badge&logo=qdrant)](https://qdrant.tech)

> [!IMPORTANT]
> **🚀 Featured Integration: MeshAPI Support**
> HighHEM now features native, out-of-the-box integration with **[MeshAPI](https://meshapi.ai)**! MeshAPI provides a robust, unified, OpenAI-compatible API to route query and embedding requests to various frontier LLMs (e.g. GPT-4o, Claude 3.5 Sonnet, Llama 3) under a single key. By combining MeshAPI's cost-effective models with HighHEM's **Semantic Highlighting**, you can achieve up to **70% cost savings** on RAG workloads!

## 🎯 Overview

This project implements a RAG (Retrieval-Augmented Generation) system optimized for efficiency, safety, and model flexibility:

1. **Semantic Highlighting** - Prunes context intelligently at the sentence level, reducing input token usage by 30-70%.
2. **HHEM Validation** - Validates response faithfulness using Vectara's Hughes Hallucination Evaluation Model (HHEM).
3. **MeshAPI Integration** - Routes requests seamlessly to alternative models (e.g. custom or next-gen models like `openai/gpt-5.4`) with zero vendor lock-in.

### Key Features

- ✅ Upload documents (PDF, MD, TXT, JSON)
- ✅ Semantic Highlighting for context pruning
- ✅ HHEM validation for hallucination detection
- ✅ Comparison endpoint (with vs without optimizations)
- ✅ Metrics tracking and cost analysis
- ✅ **MeshAPI & OpenAI-Compatible routing** - Full support for [MeshAPI](https://meshapi.ai), DeepSeek, Groq, OpenRouter, and Ollama.
- ✅ **Robust Tokenization Fallbacks** - Handles custom/experimental model names seamlessly without tokenizer errors.

### Tech Stack

- **Python 3.12**
- **FastAPI** - API framework
- **MeshAPI / OpenAI-Compatible API** (MeshAPI, DeepSeek, Groq, Ollama, etc.) - LLM & embeddings
- **LangChain** - Document parsing & chunking
- **Qdrant** - Vector store
- **Transformers** - Semantic Highlighting & HHEM models

## 📁 Project Structure

```
highhem/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app
│   ├── models.py               # Pydantic models
│   ├── config.py               # Configuration
│   ├── services/
│   │   ├── __init__.py
│   │   ├── document_processor.py  # Document parsing & chunking
│   │   ├── vector_store.py        # Qdrant operations
│   │   ├── semantic_highlighter.py # Semantic highlighting
│   │   ├── hhem_validator.py      # HHEM validation
│   │   └── rag_engine.py          # RAG orchestration
│   └── utils/
│       ├── __init__.py
│       └── metrics.py          # Metrics tracking
├── notebooks/
│   └── rag_showcase.ipynb      # Interactive tutorial (semantic highlighting + HHEM)
├── uploads/                     # Temporary file storage
├── data/                       # Qdrant data persistence
├── tests/
│   ├── __init__.py
│   └── test_api.py
├── pyproject.toml              # uv package management
├── .env.example                # Environment template
├── docker-compose.yml          # Docker compose config
└── README.md
```

## 🚀 Quick Start

### Prerequisites

- Python 3.12+
- [UV](https://docs.astral.sh/uv/) - Modern Python package manager
- Docker (optional, for Qdrant)

### 1. Clone and Setup

```bash
# Clone the repository
cd highhem

# Create virtual environment (Python 3.12) and install dependencies
uv venv --python 3.12
source .venv/bin/activate  # Windows: .venv\Scripts\activate
uv pip install -e ".[dev,notebook]"
```

### 2. Configure Environment

```bash
# Copy environment template
cp .env.example .env

# Edit .env with your OpenAI or MeshAPI credentials
OPENAI_API_KEY=your-api-key-here
OPENAI_BASE_URL=https://api.meshapi.ai/v1  # Set custom base URL for alternative providers
```

### 3. Start Qdrant

```bash
# Using Docker
docker run -d -p 6333:6333 qdrant/qdrant:latest

# Or using docker-compose
docker-compose up -d qdrant
```

### 4. Run the API

```bash
# Development mode with auto-reload
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Or using Python module
python -m app.main
```

The API will be available at `http://localhost:8000`

- API Docs: `http://localhost:8000/docs`
- Health Check: `http://localhost:8000/health`

## 📓 Jupyter Notebook — Interactive Tutorial

`notebooks/rag_showcase.ipynb` is a self-contained educational notebook that demonstrates both optimisations side-by-side with live code, visualisations, and student exercises.

### What the notebook covers

| Section | Topic |
|---|---|
| 1 | Setup — install dependencies, load models |
| 2 | Sample corpus — climate facts mixed with irrelevant noise |
| 3 | **Part 1: Semantic Highlighting** — before/after comparison, relevance score charts, threshold trade-off |
| 4 | **Part 2: HHEM Hallucination Detection** — faithful vs hallucinated answers, gauge charts, threshold sensitivity |
| 5 | Combined summary — token/cost comparison across all three modes |
| 6 | End-to-end pipeline (optional, requires Qdrant + OpenAI key) |
| 7 | Student exercises (Easy → Hard → Bonus) |

### Running the notebook

```bash
# 1. Create and activate a virtual environment (Python 3.12)
uv venv --python 3.12
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 2. Install all dependencies (including notebook extras)
uv pip install -e ".[dev,notebook]"

# 3. Copy environment config
cp .env.example .env
# Add your OPENAI_API_KEY to .env (only needed for Section 6)

# 4. Launch Jupyter
jupyter notebook notebooks/rag_showcase.ipynb
```

> **Note:** The first run downloads two HuggingFace models (~1.2 GB total). Subsequent runs use the local cache and start instantly. Sections 1–5 and all student exercises work **without** an OpenAI key or a running Qdrant instance.

---

## 📖 API Documentation

### Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Root endpoint |
| `/health` | GET | Health check |
| `/upload` | POST | Upload document (PDF, MD, TXT, JSON) |
| `/query` | POST | Query documents |
| `/compare` | POST | Compare all three modes |
| `/collection` | DELETE | Reset vector store |

### Example Usage

#### Upload a Document

```bash
curl -X POST "http://localhost:8000/upload" \
  -F "file=@document.pdf"
```

#### Query Documents

```bash
# Baseline mode (no optimizations)
curl -X POST "http://localhost:8000/query" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the main features?",
    "mode": "baseline",
    "top_k": 5
  }'

# Semantic mode (with highlighting)
curl -X POST "http://localhost:8000/query" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the main features?",
    "mode": "semantic",
    "top_k": 5
  }'

# Full mode (highlighting + HHEM)
curl -X POST "http://localhost:8000/query" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the main features?",
    "mode": "full",
    "top_k": 5
  }'
```

#### Compare All Modes

```bash
curl -X POST "http://localhost:8000/compare" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the benefits?",
    "top_k": 5
  }'
```

## 🧪 Testing

```bash
# Run the test script
uv run python tests/test_api.py

# Or using pytest
uv run pytest tests/
```

## 📊 Performance Metrics

### Expected Results

Based on typical usage with GPT-4o-mini:

| Metric | Baseline | Semantic | Full | Improvement |
|--------|----------|----------|------|-------------|
| **Input Tokens** | 2,500 | 1,200 | 1,200 | 52% reduction |
| **Cost per Query** | $0.000375 | $0.000180 | $0.000180 | 52% savings |
| **Latency** | 2.1s | 1.3s | 2.8s | 38% faster (semantic) |
| **Quality (HHEM)** | N/A | N/A | 0.85 | Validated |

### Monthly Savings (10K queries)

```
Baseline:  10,000 × $0.000375 = $3.75/month
Semantic:  10,000 × $0.000180 = $1.80/month
Full:      10,000 × $0.000180 = $1.80/month

Monthly Savings: $1.95 (52% reduction)
Annual Savings:  $23.40
```

For 100K queries/month: **$195/month savings**

## 🐳 Docker Deployment

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f api

# Stop
docker-compose down
```

## ⚙️ Configuration

All configuration is done via environment variables or `.env` file:

| Variable | Default | Description |
|----------|---------|-------------|
| `OPENAI_API_KEY` | - | OpenAI or compatible provider API key (required) |
| `OPENAI_BASE_URL` | - | Optional base URL for alternative endpoints (e.g. `https://api.meshapi.ai/v1`) |
| `QDRANT_HOST` | localhost | Qdrant host |
| `QDRANT_PORT` | 6333 | Qdrant port |
| `EMBEDDING_MODEL` | text-embedding-3-small | Embedding model name |
| `LLM_MODEL` | gpt-4o-mini | LLM model name |
| `CHUNK_SIZE` | 500 | Document chunk size |
| `CHUNK_OVERLAP` | 50 | Chunk overlap |
| `SEMANTIC_THRESHOLD` | 0.5 | Semantic highlighting threshold |
| `HHEM_THRESHOLD` | 0.5 | HHEM validation threshold |

## 🌐 Powered by MeshAPI — Unified Model Routing & Cost Efficiency

HighHEM is optimized to leverage **[MeshAPI](https://meshapi.ai)** out of the box, allowing you to access premium frontier models through a single unified endpoint. In addition, the codebase is fully compliant with any OpenAI-compatible provider (e.g. DeepSeek, Groq, OpenRouter, or local Ollama instances).

### 🔥 Why MeshAPI is the Recommended Provider:

1. **Unified Endpoint**: Query GPT-4o, Claude 3.5 Sonnet, Llama 3, and specialized models using a single unified API client.
2. **Ultra-Low Latency & High Availability**: Enjoy high uptime and automatic load balancing.
3. **Optimized Cost Performance**: Match MeshAPI's highly competitive pricing with HighHEM's **Semantic Highlighting** to get frontier LLM intelligence at a fraction of the cost.

### 🛠️ Setting up MeshAPI in HighHEM

Integrating MeshAPI takes less than 30 seconds. Simply modify the configuration variables in your local `.env` file:

```env
# 1. Provide your MeshAPI key
OPENAI_API_KEY=rsk_your_meshapi_key_here

# 2. Redirect the base URL to MeshAPI's endpoint
OPENAI_BASE_URL=https://api.meshapi.ai/v1

# 3. Specify the desired models (MeshAPI routes these dynamically)
EMBEDDING_MODEL=text-embedding-3-small
LLM_MODEL=openai/gpt-4o-mini  # Or try any other model like deepseek-chat, openai/gpt-5.4, etc.
```

### 🧠 Under the Hood: Custom Model & Tokenizer Resilience

To ensure compatibility with custom model names mapped through alternative endpoints (e.g. `openai/gpt-5.4` or `deepseek-chat`), HighHEM implements custom fail-safes:

- **Token Estimation Fallbacks**: Library-level tools (like `tiktoken`) fail when encountering non-standard OpenAI model names. HighHEM wraps tokenizer instantiation inside an intelligent fallback wrapper. If a model name is unrecognized, it automatically uses the standard `"cl100k_base"` encoding (or standard character-based approximations) to keep token counting accurate without raising errors.
- **Unified Base URL Propagation**: The codebase automatically feeds `OPENAI_BASE_URL` into all client handlers (including LangChain, embedding generation pipelines, and direct OpenAI SDK instances), ensuring consistent and complete routing to MeshAPI.

## 📝 Architecture

```
┌─────────────┐
│ Upload File │
└──────┬──────┘
       │
┌──────▼───────────────────────┐
│ 1. LangChain Document Loader │
│    - PDF, MD, TXT, JSON      │
└──────┬───────────────────────┘
       │
┌──────▼────────────────────┐
│ 2. Chunking & Embedding   │
│    - OpenAI embeddings    │
└──────┬────────────────────┘
       │
┌──────▼──────────┐
│ 3. Qdrant Store │
└──────┬──────────┘
       │
┌──────▼────────────────────────┐
│ 4. Query (3 modes)            │
│    A. Baseline (no optimizations)│
│    B. With Semantic Highlighting│
│    C. Full (Highlighting + HHEM)│
└──────┬────────────────────────┘
       │
┌──────▼─────────────────┐
│ 5. Metrics Comparison  │
│    - Token savings     │
│    - Cost analysis     │
│    - Quality scores    │
└────────────────────────┘
```

## 📄 License

MIT License

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
