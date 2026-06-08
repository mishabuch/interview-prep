# Project 1 — AI RAG Assistant · Setup Guide
> Semantic Q&A over your own documents. Upload PDFs/TXT → ask questions → get answers with source citations.

---

## Tech Stack
| Layer | Tool |
|---|---|
| API | FastAPI + Uvicorn |
| Parsing | pdfplumber |
| Embeddings | sentence-transformers |
| Vector store | ChromaDB |
| LLM | Anthropic Claude API |
| Memory | LangChain ConversationBufferWindowMemory |
| Observability | LangSmith |
| Frontend | Streamlit |
| Deploy | Hugging Face Spaces |

---

## Prerequisites
- Python 3.11 venv active (see COMMON_SETUP.md)
- `ANTHROPIC_API_KEY` from console.anthropic.com
- `LANGCHAIN_API_KEY` from smith.langchain.com

---

## Installation

```bash
cd ~/dev/projects/ai-rag-assistant
python -m venv venv && source venv/bin/activate
pip install --upgrade pip

pip install fastapi uvicorn python-dotenv anthropic
pip install pdfplumber sentence-transformers
pip install chromadb faiss-cpu
pip install langchain langchain-community langsmith
pip install streamlit
pip install pytest httpx   # testing
```

---

## Project Structure

```
ai-rag-assistant/
├── main.py                  # FastAPI app entry point
├── config.py                # Loads .env settings
├── routers/
│   ├── health.py            # GET /health
│   └── documents.py         # POST /documents/upload, GET /documents/collections
├── services/
│   ├── document_parser.py   # parse_pdf(), parse_txt(), chunk_text()
│   ├── vector_store.py      # ChromaDB client, add_chunks(), search_similar()
│   ├── rag_service.py       # query() — the full RAG pipeline
│   └── memory_service.py    # LangChain conversation memory
├── tests/
│   └── test_health.py
├── app.py                   # Streamlit frontend
├── requirements.txt
└── .env                     # never commit this
```

---

## Environment Variables

Create `.env` in project root:
```
ANTHROPIC_API_KEY=sk-ant-your-key-here
LANGCHAIN_API_KEY=ls__your-key-here
LANGCHAIN_TRACING_V2=true
CHROMA_PERSIST_DIR=./chroma_db
```

---

## Run Locally

```bash
# Terminal 1 — FastAPI backend
source venv/bin/activate
uvicorn main:app --reload --port 8000

# Terminal 2 — Streamlit frontend
source venv/bin/activate
streamlit run app.py --server.port 8501

# Test the API
curl http://localhost:8000/health
curl -F "file=@yourfile.pdf" http://localhost:8000/documents/upload
```

---

## Deployment — Hugging Face Spaces

```bash
# 1. Create new Space at huggingface.co/spaces
#    SDK: Streamlit | Visibility: Public

# 2. Freeze dependencies
pip freeze > requirements.txt

# 3. Push code (HF Spaces uses Git)
git remote add hf-space https://huggingface.co/spaces/YOUR_USERNAME/ai-rag-assistant
git push hf-space main

# 4. Set secrets in HF Space Settings (not in code):
#    ANTHROPIC_API_KEY, LANGCHAIN_API_KEY, LANGCHAIN_TRACING_V2
```

---

## Key Commands

```bash
source venv/bin/activate          # activate environment
pip freeze > requirements.txt     # save dependencies
pytest tests/                     # run tests
deactivate                        # exit venv
```
