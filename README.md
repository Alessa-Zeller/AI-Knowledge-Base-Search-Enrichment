# AI-Powered Knowledge Base Search & Enrichment

A comprehensive RAG (Retrieval-Augmented Generation) system that allows users to upload documents, search them using natural language, and get AI-generated answers with intelligent enrichment suggestions.

## Features

### Core Functionality
- **Document Upload & Storage**: Support for PDF, DOCX, and TXT files
- **Natural Language Search**: Ask questions in plain language
- **AI-Generated Answers**: Get intelligent responses using retrieved documents
- **Completeness Detection**: AI identifies when information is missing or uncertain
- **Enrichment Suggestions**: Get recommendations to improve the knowledge base

### Advanced Features
- **Structured JSON Output**: Responses include confidence levels, missing information, and enrichment suggestions
- **Auto-Enrichment**: Automatic fetching of missing data from external sources
- **Source Citations**: Detailed information about which documents were used
- **Processing Metrics**: Track processing time and performance
- **Enhanced UI**: Modern, responsive interface with real-time feedback

## Quick Start

### Backend Setup
```bash
cd backend
pip install -r requirements.txt
export OPENAI_API_KEY=your_openai_key_here
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend Setup
```bash
cd frontend
# Open index.html in your browser
# Or serve with a simple HTTP server:
python -m http.server 3000
```

## API Endpoints

- `GET /` - Health check
- `GET /health` - Detailed system status
- `POST /upload` - Upload documents (PDF, DOCX, TXT)
- `POST /search` - Search with natural language queries
- `GET /documents/count` - Get document count
- `GET /documents/list` - List all documents
- `DELETE /documents/reset` - Reset knowledge base

## Search Query Format

```json
{
  "query": "What is machine learning?",
  "top_k": 5,
  "include_auto_enrichment": true
}
```

## Response Format

```json
{
  "query": "What is machine learning?",
  "answer": "Machine learning is...",
  "confidence": "high",
  "sources": [...],
  "missing_info": [...],
  "enrichment_suggestions": [...],
  "retrieved_chunks": 5,
  "processing_time": 1.23,
  "auto_enrichment": {...},
  "reasoning": "Strong evidence found in multiple documents"
}
```

## Requirements Met

✅ **Core Requirements**
- Document Upload & Storage
- Natural Language Search
- AI-Generated Answers
- Completeness Detection
- Enrichment Suggestions

✅ **Stretch Goals**
- Structured JSON output with confidence levels
- Auto-enrichment for missing data
- Enhanced user interface
- Performance metrics and monitoring

## Design Decisions

- Retrieval-Augmented Generation (RAG): Kept the core flow simple and reliable — chunk -> embed -> retrieve -> generate. Retrieval uses ChromaDB for persistence and durability, generation uses OpenAI via LangChain for quick iteration.
- Embeddings: Chose `sentence-transformers/all-MiniLM-L6-v2` for a strong speed/quality trade-off and easy local CPU inference.
- Chunking: `RecursiveCharacterTextSplitter` with 500 chars and 100 overlap to preserve context while keeping token counts low for the LLM.
- Persistence: Chroma persistent client (`./backend/chroma_db` by default) so indexed data survives restarts.
- API: FastAPI with permissive CORS in development to simplify local testing and the static frontend.
- Output shape: Structured JSON with confidence, missing_info, and enrichment_suggestions so the UI can present rich, trustable answers.
- Auto-enrichment: Implemented a minimal, keyword-based placeholder in `app/main.py` to demonstrate the interface and UI wiring without relying on flaky external calls.
- Simplicity-first UI: Static frontend using fetch-based calls; avoids heavy build toolchains and keeps setup fast.

## Trade-offs (24h Constraint)

- Simpler embedding model: Prioritized fast local indexing over the absolute best semantic performance.
- Minimal enrichment: Stubbed enrichment rather than integrating external APIs (e.g., Wikipedia) to avoid rate limits and long debugging cycles.
- Open CORS, no auth: Eases demo and local testing; would lock down origins and add auth in production.
- Basic eval/observability: Console logs and timing only; full tracing and evaluation suite are out of scope for 24h.
- Non-streaming responses: Simpler client/server flow; streaming could be added later.
- Limited content safety: Relies on retrieval grounding and model; full red-teaming and filters are future work.

## How to Run (Local Prototype)

### Backend
```bash
cd backend
pip install -r requirements.txt
export OPENAI_API_KEY=your_openai_key_here
# optional
export OPENAI_MODEL=gpt-4o-mini
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API docs: `http://localhost:8000/docs`

### Frontend
```bash
cd frontend
python -m http.server 3000
```

Open: `http://localhost:3000`

### Optional: Docker (Backend)
```bash
cd backend
docker build -t kb-backend .
docker run --rm -e OPENAI_API_KEY=your_openai_key_here -p 8000:8000 kb-backend
```

## How to Test (cURL Examples)

Health checks:
```bash
curl -s http://localhost:8000/
curl -s http://localhost:8000/health | jq .
```

Upload a document:
```bash
curl -s -X POST \
  -F "file=@/path/to/your/doc.pdf" \
  http://localhost:8000/upload | jq .
```

Run a search (with optional auto-enrichment):
```bash
curl -s -X POST http://localhost:8000/search \
  -H "Content-Type: application/json" \
  -d '{
        "query": "What is the SLA for onboarding?",
        "top_k": 5,
        "include_auto_enrichment": true
      }' | jq .
```

Introspection endpoints:
```bash
curl -s http://localhost:8000/documents/count | jq .
curl -s http://localhost:8000/documents/list | jq .
curl -s -X DELETE http://localhost:8000/documents/reset | jq .
```

Tip: You can also try requests directly in the Swagger UI at `http://localhost:8000/docs`.
