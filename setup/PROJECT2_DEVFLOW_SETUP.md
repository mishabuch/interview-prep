# Project 2 — DevFlow · Setup Guide
> AI-powered GitHub PR code reviewer. Webhook fires on PR open → Claude reviews the diff → posts comment back on GitHub.

---

## Tech Stack
| Layer | Tool |
|---|---|
| Backend | Django + Django REST Framework |
| Database | PostgreSQL 16 |
| Auth | JWT (djangorestframework-simplejwt) |
| GitHub integration | PyGithub |
| Webhook validation | HMAC-SHA256 |
| LLM | Anthropic Claude API |
| Local testing | ngrok |
| Frontend | Streamlit dashboard |
| Deploy | Railway |

---

## Prerequisites
- Python 3.11 venv active (see COMMON_SETUP.md)
- PostgreSQL 16 running (`brew services start postgresql@16`)
- `ANTHROPIC_API_KEY` from console.anthropic.com
- A GitHub account + a test repository
- `GITHUB_TOKEN` — Personal Access Token with repo scope (github.com → Settings → Developer settings → Personal access tokens)

---

## Installation

```bash
cd ~/dev/projects/devflow
python -m venv venv && source venv/bin/activate
pip install --upgrade pip

pip install django djangorestframework
pip install psycopg2-binary python-dotenv
pip install djangorestframework-simplejwt
pip install PyGithub requests
pip install streamlit
pip install pytest pytest-django   # testing

# Start Django project
django-admin startproject devflow .
python manage.py startapp reviews
```

---

## Database Setup

```bash
# Create PostgreSQL database
createdb devflow_dev

# Run migrations after creating models
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser   # for /admin
```

---

## Project Structure

```
devflow/
├── devflow/
│   ├── settings/
│   │   ├── base.py          # shared settings
│   │   ├── local.py         # local dev (DEBUG=True)
│   │   └── production.py    # Railway deploy
│   └── urls.py
├── reviews/
│   ├── models.py            # Repo, PRReview models
│   ├── views/
│   │   ├── webhook.py       # POST /webhooks/github/ — HMAC verify + process
│   │   └── dashboard.py     # GET /api/reviews/
│   └── services/
│       └── review_service.py  # generate_review(diff) → calls Claude
├── app.py                   # Streamlit dashboard
├── requirements.txt
├── Procfile                 # for Railway: web: gunicorn devflow.wsgi
└── .env
```

---

## Environment Variables

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
GITHUB_TOKEN=ghp_your-token-here
GITHUB_WEBHOOK_SECRET=your-random-secret-string
DATABASE_URL=postgresql://localhost/devflow_dev
SECRET_KEY=your-django-secret-key
DEBUG=True
```

---

## Local Webhook Testing with ngrok

```bash
# Install ngrok
brew install ngrok

# In Terminal 1: start Django
source venv/bin/activate
python manage.py runserver 8000

# In Terminal 2: expose to internet
ngrok http 8000
# Copy the HTTPS URL e.g. https://abc123.ngrok.io

# On GitHub: repo → Settings → Webhooks → Add webhook
#   Payload URL: https://abc123.ngrok.io/webhooks/github/
#   Content type: application/json
#   Secret: (same as GITHUB_WEBHOOK_SECRET in .env)
#   Events: Pull requests

# Open a test PR → your server receives the payload
```

---

## Run Locally

```bash
source venv/bin/activate
export DJANGO_SETTINGS_MODULE=devflow.settings.local
python manage.py runserver          # API at localhost:8000
streamlit run app.py                # Dashboard at localhost:8501
```

---

## Deployment — Railway

```bash
# Install Railway CLI
npm install -g @railway/cli
railway login

# From project root
railway init       # link to Railway project
railway up         # deploy

# Set env vars in Railway dashboard:
# DATABASE_URL (Railway auto-provides PostgreSQL)
# ANTHROPIC_API_KEY, GITHUB_TOKEN, GITHUB_WEBHOOK_SECRET, SECRET_KEY, DEBUG=False
```

Create `Procfile` in project root:
```
web: gunicorn devflow.wsgi --log-file -
```

---

## Key Commands

```bash
python manage.py makemigrations     # after model changes
python manage.py migrate            # apply migrations
python manage.py shell              # Django shell for debugging
python manage.py test               # run tests
pip freeze > requirements.txt
```
