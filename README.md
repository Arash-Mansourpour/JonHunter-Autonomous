# KARYABEEEE [![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/) [![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Docker](https://img.shields.io/badge/Docker-Enabled-blue.svg)](https://www.docker.com/)

![Grafana Dashboard Screenshot](Screenshot%20(615).png)

> **Autonomous Job Application Engine** — Discover, Match, Apply, Repeat.

**KARYABEEEE** is a production-grade, fully autonomous job application engine. It's not just a script—it's a robust, observable system that continuously discovers, evaluates, and applies to high-signal job opportunities on behalf of candidates. Built with a modular, cloud-native architecture for scalability and reliability.

### Key Capabilities
- 🧭 **Continuous Scraping** of curated job platforms (e.g., JobVision, IranTalent, LinkedIn)
- 🧠 **Semantic & Rule-Based Scoring** using AI and ML models
- 🧾 **ATS-Optimized Resume Generation** tailored per opportunity
- 📝 **Hyper-Personalized Cover Letters** generated dynamically
- ⚙️ **Automated Form Submission** via Playwright for seamless applications
- 👥 **Multi-Profile Management** with custom strategies per identity
- 📊 **Follow-Up Scheduling & Analytics** to track and optimize performance
- 👀 **Full Observability** with metrics, logs, and dashboards

This repository emphasizes:
- **Modularity**: Separate concerns for scraping, intelligence, application, orchestration, and monitoring.
- **Observability**: Health checks, Prometheus metrics, Grafana dashboards, and structured logs.
- **Security**: Environment-based configs, no hard-coded secrets, least-privilege design.
- **Extensibility**: Easy to plug in new job boards, heuristics, models, or strategies.
- **Deployment-Ready**: Docker + Compose out-of-the-box; ready for CI/CD, cloud, or Kubernetes.

---

## Table of Contents
- [Architecture Overview](#architecture-overview)
- [Tech Stack & Services](#tech-stack--services)
- [Project Layout](#project-layout)
- [Quick Start (Docker-First)](#quick-start-docker-first)
- [Environment Configuration](#environment-configuration)
- [Running Components](#running-components)
- [Health, Monitoring & Observability](#health-monitoring--observability)
- [Development Workflow](#development-workflow)
- [Security Notes](#security-notes)
- [Roadmap & Status](#roadmap--status)
- [Contributing](#contributing)
- [License](#license)

---

## Architecture Overview

KARYABEEEE is structured into logical layers for maintainability and scalability:

1. **Scraping Engine**  
   Structured scrapers for platforms like JobVision, IranTalent, and LinkedIn. Uses HTTP clients/headless browsers with anti-detection (realistic headers, agents, optional proxies).

2. **Intelligence Core**  
   Powered by Google Gemini (`google-generativeai`) and ML (LightGBM, scikit-learn). Handles relevance scoring, gap identification, and tailored content generation.

3. **Application Engine**  
   Automates forms with Playwright, including resume/cover letter attachment.

4. **Profile & Account Management**  
   Manages multiple profiles with unique preferences and constraints.

5. **Persistence & State**  
   PostgreSQL for core data (jobs, applications, analytics); Redis for caching and rate limiting.

6. **Orchestration**  
   Celery workers for async tasks (scrape, match, apply); Celery Beat for scheduling.

7. **Control Plane & API**  
   FastAPI for health, admin endpoints, and integrations (e.g., bots, dashboards).

8. **Observability**  
   Prometheus for metrics, Grafana for dashboards, structured JSON logs for ELK/Sentry.

For details, see [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

---

## Tech Stack & Services

Defined in [`docker-compose.yml`](docker-compose.yml):

| Service        | Description                                                                 | Port(s)      |
|----------------|-----------------------------------------------------------------------------|--------------|
| **app**       | FastAPI app (`karyabee.api.main:app`) with health/metrics.                  | 8000        |
| **celery_worker** | Celery workers for tasks (scrape, score, apply).                           | -           |
| **celery_beat** | Scheduler for periodic jobs (scraping, follow-ups).                        | -           |
| **db**        | PostgreSQL 15 for persistent storage.                                      | 5432 (internal) / 5433 (host) |
| **redis**     | Redis 7 for caching and rate limiting.                                     | 6379        |
| **rabbitmq**  | Message broker for Celery (UI at `/`).                                     | 5672 / 15672 |
| **prometheus**| Scrapes metrics; config in `monitoring/prometheus/prometheus.yml`.        | 9090        |
| **grafana**   | Dashboards; provisioning in `monitoring/grafana/provisioning`.             | 3001        |
| **nginx**     | Reverse proxy to app; config in `nginx/nginx.conf`.                        | 80          |

All on `karyabee_net` network.

---

## Project Layout

```
.
├── docker-compose.yml      # Stack definition
├── Dockerfile              # Python 3.11 base image
├── requirements.txt        # Pinned dependencies
├── setup.sh                # Bootstrap script
├── Screenshot (615).png    # Grafana dashboard screenshot
├── karyabee/               # Core Python package
│   ├── __init__.py
│   ├── config.py
│   ├── db.py
│   ├── logging_config.py
│   ├── metrics.py
│   ├── models.py
│   ├── schemas.py
│   ├── ai/                 # AI modules
│   │   ├── gemini_client.py
│   │   ├── matching.py
│   │   ├── resume_builder.py
│   │   ├── cover_letter.py
│   │   └── interview_predictor.py
│   ├── scrapers/           # Platform scrapers
│   │   ├── base.py
│   │   ├── jobvision.py
│   │   ├── irantalent.py
│   │   └── linkedin.py
│   ├── applier/            # Application automation
│   │   ├── base.py
│   │   └── playwright_engine.py
│   ├── orchestration/      # Celery tasks
│   │   ├── celery_app.py
│   │   └── tasks.py
│   ├── api/                # FastAPI endpoints
│   │   └── main.py
│   ├── bot/                # Notification bots
│   │   └── telegram_bot.py
│   └── monitoring/         # Health & metrics
│       ├── health.py
│       └── prometheus_exporter.py
├── tests/                  # Unit/integration tests
├── docs/                   # Documentation
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   └── OPERATIONS.md
├── env/                    # Environment (gitignore'd)
│   └── .env
├── monitoring/             # Observability configs
│   ├── prometheus/
│   └── grafana/
├── nginx/                  # Proxy configs
└── logs/                   # Mounted logs (gitignore'd)
```

---

## Quick Start (Docker-First)

**Prerequisites**: Docker, Docker Compose v2, `env/.env` configured.

1. **Setup Environment**  
   ```bash
   mkdir -p env
   cp env/.env.example env/.env  # Edit with your secrets
   ```

2. **Build & Launch**  
   ```bash
   docker-compose up -d --build
   ```

3. **Verify**  
   - API: `curl http://localhost:8000/api/health`  
   - Nginx: `http://localhost/`  
   - RabbitMQ: `http://localhost:15672` (guest/guest)  
   - Prometheus: `http://localhost:9090`  
   - Grafana: `http://localhost:3001`  
   - DB: `psql -h localhost -p 5433 -U <user> -d <db>`

4. **Bootstrap (Optional)**  
   ```bash
   chmod +x setup.sh
   ./setup.sh
   ```

---

## Environment Configuration

Use environment variables (loaded from `.env`):

| Category       | Examples                          | Description                  |
|----------------|-----------------------------------|------------------------------|
| **Core**      | `APP_ENV`, `LOG_LEVEL`           | Environment & logging.      |
| **Database**  | `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_HOST`, `POSTGRES_PORT` | DB credentials.             |
| **Broker/Cache** | `RABBITMQ_DEFAULT_USER`, `RABBITMQ_DEFAULT_PASS`, `REDIS_URL` | Messaging & caching.        |
| **AI/Services**| `GEMINI_API_KEY`                 | API keys for Gemini, etc.   |
| **Notifications** | `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` | Bot configs.                |
| **Scraping**  | Proxy URLs, rate limits          | Anti-detection settings.    |

**Warning**: Never commit secrets—use `.env` or secret managers.

---

## Running Components

- **Logs**: `docker-compose logs -f <service>` (e.g., `app`, `celery_worker`)  
- **Restart**: `docker-compose restart app celery_worker celery_beat`  
- **Teardown**: `docker-compose down -v`

---

## Health, Monitoring & Observability

- **Health Check**: `GET /api/health`  
- **Docs**: `GET /api/docs` (OpenAPI)  
- **Metrics**: Exposed via Prometheus (e.g., job discovery rate, application success).  
- **Dashboards**: Grafana with pre-configured views for rates, latencies, errors.  
- **Logs**: Structured JSON; tailed via Docker or mounted to `logs/`.

Example Metrics:
- `job_discovery_rate`  
- `application_success_total`  
- `task_latency_seconds`  

Here's an example of a Grafana dashboard in action.

---

## Development Workflow

For local dev:

1. **Virtual Env**: `python -m venv .venv && source .venv/bin/activate`  
2. **Install**: `pip install -r requirements.txt`  
3. **Run App**: `uvicorn karyabee.api.main:app --reload --host 0.0.0.0 --port 8000`  
4. **Celery Worker**: `celery -A karyabee.orchestration.celery_app.celery_app worker --loglevel=INFO`  
5. **Celery Beat**: `celery -A karyabee.orchestration.celery_app.celery_app beat --loglevel=INFO`

Ensure local services (Postgres, Redis, RabbitMQ) are running.

---

## Security Notes

- **No Hard-Coded Secrets**: All via env vars.  
- **Data Protection**: Encrypt sensitive data; avoid logging PII.  
- **Best Practices**: Use dedicated accounts, respect ToS, implement rate limits to prevent abuse.

---

## Roadmap & Status

- [x] Containerized stack (Docker + Compose)  
- [x] Config, logging, metrics setup  
- [x] Database models & scrapers  
- [x] Orchestration skeleton  
- [ ] Full auto-apply with fallbacks  
- [ ] Advanced Grafana dashboards/alerts  
- [ ] Enhanced scoring, A/B testing, predictors  
- [ ] Analytics & reporting  

---

## Contributing

Contributions welcome! Please:
- Fork the repo.  
- Create a feature branch.  
- Submit a PR with clear descriptions.  
- Follow code style (PEP8, mypy).  

See `CONTRIBUTING.md` for details (if added).

---

## License

Apache License 2.0. See [`LICENSE`](LICENSE) for details.  
Developed with ❤️ by me.
