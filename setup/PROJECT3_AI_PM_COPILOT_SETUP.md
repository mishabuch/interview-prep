# Project 3 — AI PM Copilot · Setup Guide
> Multi-agent project planning tool. Input a feature description → 3 AI agents generate user stories, validate them, and estimate effort. Export to Jira.

---

## Tech Stack
| Layer | Tool |
|---|---|
| Backend | Django + Django REST Framework |
| Database | PostgreSQL 16 |
| Auth | JWT (djangorestframework-simplejwt) |
| Multi-agent | LangGraph (StateGraph) |
| LLM | Anthropic Claude API (via langchain-anthropic) |
| Jira export | Jira REST API v3 |
| Frontend | Streamlit |
| Deploy | Render |

---

## The 3 Agents
| Agent | Input | Output |
|---|---|---|
| **Planner** | Feature description | 3–5 epics + 2–4 user stories per epic |
| **Validator** | Raw user stories | Improved stories with acceptance criteria |
| **Estimator** | Validated stories | Story points (1/2/3/5/8) with reasoning |

Orchestrated by a **LangGraph StateGraph**: Planner → Validator → Estimator → END. Validation errors loop back to Planner (max 2 retries).

---

## Prerequisites
- Python 3.11 venv active (see COMMON_SETUP.md)
- PostgreSQL 16 running
- `ANTHROPIC_API_KEY` from console.anthropic.com
- Optional: Jira account at atlassian.com/free for export testing

---

## Installation

```bash
cd ~/dev/projects/ai-pm-copilot
python -m venv venv && source venv/bin/activate
pip install --upgrade pip

pip install django djangorestframework
pip install psycopg2-binary python-dotenv
pip install djangorestframework-simplejwt
pip install langgraph langchain-anthropic langchain
pip install streamlit
pip install requests    # for Jira API export
pip install pytest pytest-django
```

---

## Database Setup

```bash
createdb pm_copilot_dev
python manage.py startapp stories
python manage.py makemigrations
python manage.py migrate
```

---

## Project Structure

```
ai-pm-copilot/
├── pm_copilot/
│   ├── settings/
│   │   ├── base.py
│   │   ├── local.py
│   │   └── production.py
│   └── urls.py
├── stories/
│   ├── models.py            # Project, Feature, UserStory models
│   ├── views/
│   │   ├── features.py      # POST /api/features/, GET /api/features/{id}/
│   │   └── stories.py       # PUT /api/stories/{id}/, POST /api/stories/{id}/regenerate
│   └── serializers.py
├── agents/
│   ├── planner.py           # LangGraph node: feature → epics → stories
│   ├── validator.py         # LangGraph node: stories → validated + acceptance criteria
│   ├── estimator.py         # LangGraph node: stories → story points
│   └── orchestrator.py      # StateGraph wiring all 3 agents
├── services/
│   └── export_service.py    # jira_format(story) → dict for Jira API
├── app.py                   # Streamlit frontend
├── requirements.txt
└── .env
```

---

## Environment Variables

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
DATABASE_URL=postgresql://localhost/pm_copilot_dev
SECRET_KEY=your-django-secret-key
DEBUG=True
JIRA_BASE_URL=https://yoursite.atlassian.net    # optional
JIRA_EMAIL=your@email.com                       # optional
JIRA_API_TOKEN=your-jira-token                  # optional
```

---

## Run Locally

```bash
source venv/bin/activate
export DJANGO_SETTINGS_MODULE=pm_copilot.settings.local
python manage.py runserver          # API at localhost:8000
streamlit run app.py                # Frontend at localhost:8501

# Test the pipeline end-to-end
curl -X POST http://localhost:8000/api/features/ \
  -H "Content-Type: application/json" \
  -d '{"title": "User notifications", "description": "Users should receive email alerts when their order ships"}'
```

---

## Deployment — Render

```bash
# 1. Push code to GitHub (render deploys from GitHub)
# 2. render.com → New Web Service → connect your repo
# 3. Build command: pip install -r requirements.txt
# 4. Start command: gunicorn pm_copilot.wsgi
# 5. Add PostgreSQL: render.com → New → PostgreSQL (free tier)
# 6. Set env vars in Render dashboard
```

---

## Key Commands

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py test
pip freeze > requirements.txt
```
