# geo-analytics-service

Python microservice for GEO (Generative Engine Optimization) analysis — collects Google Search Console data, detects AI Overview presence via CTR anomaly, and computes GEO metrics for the [GEO Intelligence](https://github.com/marcosalvesdev/seach-flow-api) platform.

[![Python](https://img.shields.io/badge/python-3.11-blue)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110-009688?logo=fastapi)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Tests](https://img.shields.io/badge/tests-107%20passing-brightgreen)](./tests)
[![Coverage](https://img.shields.io/badge/coverage-80%25+-brightgreen)](#testing)

---

## Overview

This service is the data science layer of the GEO Intelligence SaaS platform. While the main [NestJS backend](https://github.com/marcosalvesdev/seach-flow-api) handles authentication, multi-tenancy, and AI recommendations, this Python service is responsible for:

- Connecting to **Google Search Console** via OAuth 2.0 and syncing query performance data
- **Detecting AI Overview presence** through CTR anomaly detection (no direct Google signal required)
- Computing the **GEO Index** — a 0–100 score representing organic visibility health against generative AI
- Computing **Share of Voice** per AI brand (ChatGPT, Gemini, Copilot)
- Storing OAuth tokens **encrypted at rest** using Fernet (AES-256)
- Full **multi-tenant** data isolation per `organization_id`

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI + Uvicorn |
| Database | PostgreSQL 16 (SQLAlchemy ORM) |
| Migrations | Alembic |
| Encryption | Fernet (AES-256) |
| Testing | pytest + SQLite in-memory |
| Infrastructure | Docker Compose |
| External API | Google Search Console API v3 |
| Auth | OAuth 2.0 (Google) |

## Architecture

```
src/
├── api/          # HTTP layer — FastAPI routers and route aggregation
├── analytics/    # Core algorithms: AIO detection, GEO Index, Share of Voice, time series
├── cleaning/     # Data normalization before analysis
├── collection/   # Google Search Console integration
├── core/         # Config, security (API key auth), logging, exceptions
├── middleware/   # CORS and HTTP error handling
├── models/       # SQLAlchemy models, repository pattern, Pydantic schemas
├── services/     # Business logic orchestration
└── utils/        # URL validators, formatters, Fernet encryption wrapper
```

**Request flow:**

```
POST /api/v1/gsc/sync
  └─> GSCService.sync()
        ├─> OAuth token decrypt
        ├─> GSC API collection
        ├─> Data cleaning
        └─> GSCRepository.save_rows()

GET /api/v1/analytics/*
  └─> Fetch rows by org + site
        ├─> AIO detection (CTR anomaly)
        ├─> GEO Index calculation
        └─> Return JSON metrics
```

## How the Algorithms Work

### AI Overview Detection

Google does not expose when an AI Overview is shown. This service infers it by comparing the **actual CTR** against the **expected CTR** for each organic position:

| Position | Expected CTR |
|---|---|
| 1st | ~28% |
| 2nd | ~15% |
| 3rd | ~11% |

A page ranking 1st with only 8% CTR is flagged as AIO-suppressed — the generative result is absorbing clicks that would otherwise go to organic links.

### GEO Index

A 0–100 score representing how much generative AI is suppressing organic visibility:

```
GEO Index = mean(actual_ctr / expected_ctr) × 100
```

- **100** — no AIO impact detected
- **0** — CTR fully suppressed across all monitored queries

### Share of Voice

Measures what percentage of clicks each AI engine (ChatGPT, Gemini, Copilot) captures within the monitored queries.

## API Reference

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| GET | `/` | — | Health check |
| POST | `/api/v1/gsc/sync` | API Key | Trigger GSC data sync |
| POST | `/api/v1/oauth/token` | API Key | Store encrypted OAuth token |
| GET | `/api/v1/oauth/status` | API Key | Check if token exists |
| DELETE | `/api/v1/oauth/token` | API Key | Remove OAuth token |
| GET | `/api/v1/analytics/geo-metrics` | API Key | GEO Index and AIO impact |
| GET | `/api/v1/analytics/share-of-voice` | API Key | Share of Voice by AI brand |

Interactive docs available at `http://localhost:8000/docs` when running locally.

## Getting Started

**Prerequisites:** Python 3.11+, Docker

```bash
git clone https://github.com/Guilherme-fonte/geo-analytics-service.git
cd geo-analytics-service

python -m venv venv && source venv/bin/activate  # Linux/macOS
# python -m venv venv && venv\Scripts\activate  # Windows

pip install -r requirements.txt

cp .env.example .env
# Fill in DATABASE_URL, ENCRYPTION_KEY, and optionally API_KEY

docker-compose up -d          # Start PostgreSQL
python -m alembic upgrade head  # Run migrations
uvicorn src.main:app --reload --port 8000
```

## Testing

107 tests, all running against an in-memory SQLite database. No Docker required for tests.

```bash
pytest tests/ -v
pytest tests/ --cov=src --cov-report=term-missing
```

External integrations (Google API, PostgreSQL) are fully mocked.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `ENCRYPTION_KEY` | Yes | Fernet key for encrypting OAuth tokens |
| `API_KEY` | No | API key for protected routes (disabled if empty) |
| `LOG_LEVEL` | No | `DEBUG`, `INFO`, `WARNING`, or `ERROR` |

See [.env.example](./.env.example) for the full reference.

## Context

This service was built as part of an internship project — a real B2B SaaS platform called **GEO Intelligence**. The full platform includes a [NestJS backend](https://github.com/marcosalvesdev/seach-flow-api) handling multi-tenancy, OAuth flows with the main frontend, and AI-powered content recommendations via the Anthropic API.

Built by [Guilherme Andrade](https://github.com/Guilherme-fonte) · [LinkedIn](https://linkedin.com/in/guilherme-andrade-5a9504200)
