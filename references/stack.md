# Tech Stack — Full Version Reference

## 1.1 Runtime & Language

| Component | Version | Description |
|-----------|---------|-------------|
| **Python** | ^3.11 | Primary backend language |
| **Poetry** | ^2.1 | Dependency manager and virtual envs |
| **pyenv** | latest | Python version manager |

## 1.2 Web Framework

| Component | Version | Description |
|-----------|---------|-------------|
| **FastAPI** | ^0.104.1 | High-performance async web framework |
| **Uvicorn** | ^0.34.0 | ASGI server for production |
| **Pydantic** | ^2.4.2 | Data validation and serialization |
| **pydantic-settings** | ^2.9.1 | Config from environment variables |

## 1.3 Database

| Component | Version | Description |
|-----------|---------|-------------|
| **PostgreSQL** | 15+ | Primary relational database |
| **SQLAlchemy** | ^2.0.23 | Full async ORM |
| **SQLModel** | ^0.0.24 | SQLAlchemy + Pydantic integration |
| **asyncpg** | ^0.30.0 | Async PostgreSQL driver |
| **dbmate** | latest | Database migrations |

## 1.4 Cache & Message Queue

| Component | Version | Description |
|-----------|---------|-------------|
| **Redis** | ^6.2.0 | Cache, message buffer, Celery broker |
| **Celery** | ^5.5.3 | Async tasks and scheduled jobs |

## 1.5 Observability

| Component | Version | Description |
|-----------|---------|-------------|
| **Sentry SDK** | ^2.39.0 | Error and performance monitoring |
| **Loguru** | ^0.7.2 | Structured logging |
| **OpenTelemetry** | ^1.22.0 | Metrics and distributed tracing |

## 1.6 Testing & Quality

| Component | Version | Description |
|-----------|---------|-------------|
| **pytest** | ^7.4.2 | Testing framework |
| **pytest-asyncio** | ^0.21.1 | Async support for tests |
| **pytest-cov** | ^5.0.0 | Test coverage |
| **pylint** | ^3.3.6 | Static linter |
| **isort** | ^6.0.1 | Import ordering |
| **pre-commit** | ^3.4.0 | Pre-commit hooks |

## 1.7 HTTP & Networking

| Component | Version | Description |
|-----------|---------|-------------|
| **httpx** | ^0.25.0 | Async HTTP client |
| **requests** | ^2.31.0 | Sync HTTP client (legacy) |

## 1.8 Utilities

| Component | Version | Description |
|-----------|---------|-------------|
| **python-dotenv** | ^1.0.0 | Load env variables |
| **python-jose** | ^3.4.0 | JWT tokens (with cryptography) |
| **pytz** | ^2025.1 | Timezone handling |
| **backoff** | ^2.2.1 | Retry with exponential backoff |
| **cachetools** | ^5.5.0 | In-memory cache utilities |
