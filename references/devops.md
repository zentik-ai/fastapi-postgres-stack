# DevOps, Testing & Tooling Reference

## Testing

### Structure

```
tests/
├── conftest.py              # Shared fixtures
├── api/
│   └── test_{domain}.py    # Endpoint tests
├── db/
│   └── test_{entity}.py    # Repository tests
└── services/
    └── test_{service}.py   # Service tests
```

### Common Fixtures (tests/conftest.py)

```python
import pytest
from httpx import AsyncClient
from app.main import app
from app.db.database import get_db

@pytest.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.fixture
async def db_session():
    async with AsyncSessionLocal() as session:
        yield session
        await session.rollback()
```

### Running Tests

```bash
pytest                                       # All tests
pytest --cov=app --cov-report=html          # With coverage
pytest tests/api/test_messaging.py -v       # Specific tests
pytest -v --asyncio-mode=auto               # Async tests
```

## Celery Async Tasks

### Task Structure (app/celery/tasks.py)

```python
from app.celery_app import celery
from loguru import logger

@celery.task
def my_sync_task(param1: str, param2: int):
    logger.info(f"Processing task with {param1}, {param2}")
    # Task logic

@celery.task
def my_async_wrapper_task(param: str):
    """Wrapper to run async code in Celery."""
    import asyncio, nest_asyncio
    nest_asyncio.apply()
    loop = asyncio.get_event_loop()
    return loop.run_until_complete(_my_async_task(param))

async def _my_async_task(param: str):
    async with AsyncSessionLocal() as db:
        # Async operations
        pass
```

### Scheduled Tasks (Beat)

```python
# app/celery_app.py
from celery.schedules import crontab

celery.conf.beat_schedule = {
    "task-every-5-minutes": {
        "task": "app.celery.tasks.my_task",
        "schedule": crontab(minute="*/5"),
    },
    "task-daily-at-midnight": {
        "task": "app.celery.tasks.daily_task",
        "schedule": crontab(hour=0, minute=0),
    },
}
```

### Running Celery

```bash
# Worker + Beat in one process (development)
celery -A app.celery_app worker --beat --loglevel=info

# Separate worker and beat (production)
celery -A app.celery_app worker --loglevel=info
celery -A app.celery_app beat --loglevel=info
```

## Docker

### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install poetry
COPY pyproject.toml poetry.lock ./
RUN poetry config virtualenvs.create false \
    && poetry install --no-dev --no-interaction --no-ansi
COPY app ./app
COPY entrypoint.sh ./
RUN chmod +x entrypoint.sh
EXPOSE 8080
ENTRYPOINT ["./entrypoint.sh"]
```

### Docker Compose (Development)

```yaml
version: '3.9'
services:
  redis:
    image: redis:latest
    ports: ["6379:6379"]
    networks: [backend]

  database:
    build: { context: ., dockerfile: docker/db/Dockerfile }
    ports: ["5432:5432"]
    environment:
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    healthcheck:
      test: ['CMD', 'pg_isready', '-U', '${DB_USERNAME}']
      interval: 5s
      retries: 5
    networks: [backend]

  dbmate:
    build: { context: ., dockerfile: docker/dbmate/Dockerfile }
    environment:
      - DATABASE_URL=postgres://${DB_USERNAME}:${DB_PASSWORD}@database:5432/${DB_NAME}?sslmode=disable
    volumes: ["./db/migrations:/db/migrations"]
    depends_on: { database: { condition: service_healthy } }
    networks: [backend]

networks:
  backend:
    driver: bridge
```

## Makefile Commands

```makefile
install:
    poetry install --no-root
    poetry run pre-commit install

run:
    poetry run uvicorn app.main:app --reload --port 8080

run-celery:
    celery -A app.celery_app worker --beat --loglevel=info

test:
    poetry run pytest --cov=app

lint:
    poetry run pylint app/
    poetry run isort --check-only app/

format:
    poetry run isort app/
    poetry run black app/

compose-start:
    docker compose up -d

compose-stop:
    docker compose down
```

## Rate Limiting (Redis)

```python
from fastapi import Request, HTTPException
import redis

async def rate_limit(request: Request, limit: int = 100, window: int = 60):
    key = f"rate_limit:{request.client.host}"
    current = r.incr(key)
    if current == 1:
        r.expire(key, window)
    if current > limit:
        raise HTTPException(status_code=429, detail="Too many requests")
```

## API Documentation (OpenAPI)

FastAPI auto-generates:
- **Swagger UI**: `http://localhost:8080/docs`
- **ReDoc**: `http://localhost:8080/redoc`
- **OpenAPI JSON**: `http://localhost:8080/api/v1/openapi.json`

## New Project Checklist

### Initial Setup
- [ ] Create repo with folder structure
- [ ] Configure `pyproject.toml` with dependencies
- [ ] Create `docker-compose.yml` for local dev
- [ ] Configure `dbmate.yaml` for migrations
- [ ] Create `.env.example` with required variables
- [ ] Configure `pre-commit` hooks
- [ ] Create `Makefile` with useful commands

### Development
- [ ] Implement `BaseTable` for CRUD operations
- [ ] Configure logging with Loguru
- [ ] Implement auth and error handling middlewares
- [ ] Create initial database migrations
- [ ] Configure Celery if async tasks needed
- [ ] Implement health check endpoint

### Production
- [ ] Configure Sentry for error monitoring
- [ ] Create optimized `Dockerfile`
- [ ] Configure production environment variables
- [ ] Implement rate limiting
- [ ] Configure CORS appropriately
- [ ] Document API with OpenAPI
