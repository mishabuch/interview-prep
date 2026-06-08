# Project 4 — ManuPilot · Setup Guide
> AI predictive maintenance platform. Real-time sensor ingestion → anomaly detection (PyOD) → NL interface ("Why is Machine 3 alerting?") → auto-generated maintenance tickets.

---

## Tech Stack
| Layer | Tool |
|---|---|
| API | FastAPI + Uvicorn |
| Database | PostgreSQL 16 + SQLAlchemy + Alembic |
| Anomaly detection | PyOD (IsolationForest) |
| Forecasting | Prophet (optional) |
| ML / numerics | pandas, numpy, scikit-learn |
| RAG over manuals | LangChain + ChromaDB + sentence-transformers |
| PDF parsing | pdfplumber |
| LLM | Anthropic Claude API |
| Charts | Plotly |
| Frontend | Streamlit |
| Containerisation | Docker + docker-compose |
| Deploy | Railway (API) + Hugging Face Spaces (dashboard) |

---

## Prerequisites
- Python 3.11 venv active (see COMMON_SETUP.md)
- PostgreSQL 16 running
- Docker Desktop running
- `ANTHROPIC_API_KEY` from console.anthropic.com

---

## Installation

```bash
cd ~/dev/projects/manupilot
python -m venv venv && source venv/bin/activate
pip install --upgrade pip

pip install fastapi uvicorn python-dotenv
pip install sqlalchemy alembic psycopg2-binary

pip install pyod pandas numpy scikit-learn
pip install prophet                  # time-series forecasting (optional)

pip install langchain langchain-community
pip install chromadb sentence-transformers pdfplumber

pip install plotly streamlit
pip install anthropic

pip install pytest httpx             # testing
```

---

## Database Setup

```bash
createdb manupilot_dev

# Alembic for DB migrations
alembic init alembic
# Edit alembic/env.py to point to your DATABASE_URL
alembic revision --autogenerate -m "initial"
alembic upgrade head
```

---

## Project Structure

```
manupilot/
├── main.py                      # FastAPI entry point
├── database/
│   ├── models.py                # Machine, SensorReading, Anomaly, MaintenanceTicket
│   ├── session.py               # SQLAlchemy session factory
│   └── seed.py                  # seed initial machines
├── routers/
│   ├── ingest.py                # POST /ingest/reading, POST /ingest/batch
│   ├── machines.py              # GET /machines/, GET /machines/{id}/readings
│   ├── anomalies.py             # GET /anomalies/
│   └── tickets.py               # GET /tickets/, POST /tickets/
├── services/
│   ├── anomaly_detector.py      # PyOD IsolationForest: train + predict
│   ├── health_service.py        # health score (0–100) per machine
│   ├── manual_rag.py            # ChromaDB RAG over maintenance PDFs
│   ├── nl_interface.py          # handle_query(question) → Claude answer
│   └── ticket_service.py        # auto-generate ticket on anomaly
├── scripts/
│   └── simulate_sensors.py      # generate 24hrs of sensor data for 5 machines
├── data/
│   └── manuals/                 # maintenance PDF files go here
├── app.py                       # Streamlit dashboard
├── docker-compose.yml           # local: web + db + streamlit
├── Dockerfile
├── requirements.txt
└── .env
```

---

## Environment Variables

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
DATABASE_URL=postgresql://localhost/manupilot_dev
CHROMA_PERSIST_DIR=./chroma_db
DEBUG=True
```

---

## Run Locally

```bash
# Option A: directly
source venv/bin/activate
uvicorn main:app --reload --port 8000    # API
streamlit run app.py                     # Dashboard at :8501

# Seed simulated data
python scripts/simulate_sensors.py

# Option B: via Docker
docker-compose up --build
# API at localhost:8000 | Dashboard at localhost:8501 | DB at localhost:5432
```

---

## docker-compose.yml

```yaml
version: "3.8"
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: manupilot_dev
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports: ["5432:5432"]

  web:
    build: .
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload
    volumes: [".:/app"]
    ports: ["8000:8000"]
    env_file: .env
    depends_on: [db]

  dashboard:
    build: .
    command: streamlit run app.py --server.port 8501 --server.address 0.0.0.0
    volumes: [".:/app"]
    ports: ["8501:8501"]
    env_file: .env
    depends_on: [web]
```

---

## Deployment

```bash
# FastAPI → Railway
npm install -g @railway/cli
railway login && railway init && railway up
# Set DATABASE_URL (Railway PostgreSQL) + ANTHROPIC_API_KEY in Railway dashboard

# Streamlit dashboard → Hugging Face Spaces
# Create Space: SDK=Streamlit, Visibility=Public
# Set ANTHROPIC_API_KEY + DATABASE_URL as Space secrets
# Push: git remote add hf-space https://huggingface.co/spaces/USERNAME/manupilot
#        git push hf-space main
```

---

## Key Commands

```bash
alembic revision --autogenerate -m "description"   # new migration
alembic upgrade head                               # apply migrations
alembic downgrade -1                               # rollback one step
python scripts/simulate_sensors.py                 # seed sensor data
pytest tests/
pip freeze > requirements.txt
```
