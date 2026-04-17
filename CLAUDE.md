# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RAG system demonstrating cost savings via semantic highlighting and quality improvements via HHEM hallucination validation. Implements three query modes:
- **baseline**: Standard RAG without optimizations
- **semantic**: RAG with semantic highlighting for context pruning
- **full**: Semantic highlighting + HHEM validation

## Development Commands

### Setup
```bash
# Install dependencies (requires uv)
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[dev,notebook]"

# Configure environment
cp .env.example .env
# Add OPENAI_API_KEY to .env
```

### Running Services
```bash
# Start Qdrant (required for vector storage)
docker run -d -p 6333:6333 qdrant/qdrant:latest
# Or: make start-qdrant

# Run FastAPI development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
# Or: make run
```

### Testing
```bash
# Run API tests (requires running Qdrant and API server)
uv run python tests/test_api.py
# Or: make test
```

### Docker
```bash
make docker-up    # Start all services
make docker-down  # Stop all services
```

## Architecture

### Service Layer (`app/services/`)

Five core services orchestrated by `RAGEngine`:

1. **DocumentProcessor**: Uses LangChain loaders (PyPDFLoader, TextLoader, UnstructuredMarkdownLoader) to parse documents and RecursiveCharacterTextSplitter for chunking.

2. **VectorStore**: Qdrant integration with OpenAI embeddings (`text-embedding-3-small`). Handles collection creation, document upsert, and semantic search.

3. **SemanticHighlighter**: Wraps `zilliz/semantic-highlight-bilingual-v1` model to prune irrelevant sentences from retrieved context based on query relevance scores.

4. **HHEMValidator**: Wraps `vectara/hallucination_evaluation_model` to detect hallucinations. Returns score (0-1) and boolean flag based on threshold.

5. **RAGEngine**: Main orchestrator implementing `query_baseline()`, `query_semantic()`, and `query_full()`. Calculates token counts, cost estimates (GPT-4o-mini pricing), and execution time metrics.

### Compatibility Patches (`app/utils/compat_patches.py`)

Critical patches required for transformers 5.x compatibility with HuggingFace remote code models:

1. **XLMRobertaTokenizer patch**: Adds `build_inputs_with_special_tokens` method to XLMRobertaTokenizerFast (Rust delegation removed in 5.x).

2. **Tied weights compatibility patch**: Fixes `all_tied_weights_keys` property for remote code models using older `_tied_weights_keys` API.

3. **HHEM weight tying fix**: Calls `model.tie_weights()` after loading to fix zero-initialized `encoder.embed_tokens.weight` in T5-based models.

Patches must be applied before model load via `apply_all_patches()`. Already called in `semantic_highlighter.py` and `hhem_validator.py`.

### Configuration (`app/config.py`)

All settings via environment variables or `.env`:
- `OPENAI_API_KEY` (required)
- `QDRANT_HOST`, `QDRANT_PORT`
- `EMBEDDING_MODEL`, `LLM_MODEL`
- `CHUNK_SIZE`, `CHUNK_OVERLAP`
- `SEMANTIC_THRESHOLD`, `HHEM_THRESHOLD`

### API Endpoints (`app/main.py`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check with Qdrant status |
| `/upload` | POST | Upload document (PDF, MD, TXT, JSON) |
| `/query` | POST | Query with specified mode |
| `/compare` | POST | Compare all three modes |
| `/collection` | DELETE | Reset vector store |

## Important Notes

- **Python 3.12 required**: Codebase uses Pydantic 2 and modern Python features
- **Torch 2.6.0+ required**: torch 2.5.1 has no macOS ARM64 wheels
- **Cost tracking**: GPT-4o-mini pricing: $0.150/1M input tokens, $0.600/1M output tokens
- **First run**: Downloads ~1.2GB of HuggingFace models; cached locally for subsequent runs
- **Jupyter notebook**: `notebooks/rag_showcase.ipynb` is self-contained for sections 1-5 (no API key or Qdrant required)
