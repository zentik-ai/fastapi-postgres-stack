---
name: fastapi-postgres-stack
description: Architecture guide and best practices for working with FastAPI + PostgreSQL. Use when building new API endpoints, adding domains, writing database models, creating Celery tasks, setting up middlewares, writing tests, or following project conventions. Triggers on tasks involving FastAPI routes, SQLModel/SQLAlchemy models, Pydantic schemas, database migrations (dbmate), async patterns, or any backend development within this stack.
---

# FastAPI + PostgreSQL

## Tech Stack

- **Runtime**: Python ^3.11 + Poetry ^2.1
- **Web**: FastAPI ^0.104.1, Uvicorn ^0.34.0
- **Validation**: Pydantic ^2.4.2, pydantic-settings ^2.9.1
- **Database**: PostgreSQL 15+, SQLAlchemy ^2.0.23, SQLModel ^0.0.24, asyncpg ^0.30.0
- **Migrations**: dbmate (latest)
- **Cache/Queue**: Redis ^6.2.0, Celery ^5.5.3
- **Observability**: Sentry ^2.39.0, Loguru ^0.7.2, OpenTelemetry ^1.22.0
- **Testing**: pytest ^7.4.2, pytest-asyncio ^0.21.1, pytest-cov ^5.0.0
- **HTTP clients**: httpx ^0.25.0 (async), requests ^2.31.0 (sync)
- **Auth**: python-jose ^3.4.0
- **Utils**: python-dotenv ^1.0.0, pytz ^2025.1, backoff ^2.2.1, cachetools ^5.5.0

For full version table see [references/stack.md](references/stack.md).

## Layered Architecture

```
HTTP Request
    │
    ▼
PRESENTATION LAYER   app/api/{domain}/points.py   (FastAPI Routers)
    │  Receives HTTP, validates input via Pydantic, delegates to service
    ▼
BUSINESS LAYER       app/api/{domain}/service.py  (Services)
    │  Business logic, orchestration, validations
    ▼
DATA ACCESS LAYER    app/db/{entity}/table.py      (Repositories)
    │  CRUD ops, complex queries, DB abstraction
    ▼
DATABASE LAYER       PostgreSQL + SQLAlchemy/SQLModel
```

## Key Conventions

- **Files**: `snake_case` — `user_service.py`, `auth_middleware.py`
- **Classes**: `PascalCase` — `UserService`, `MessagesTable`
- **Functions/Methods/Variables**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE` — `MAX_RETRIES`, `DEFAULT_TIMEOUT`
- **Domain modules**: singular snake_case — `messaging/`, `webhook/`
- **DB tables**: plural snake_case — `messages`, `webhooks`

## Reference Files

Load only what is relevant to the current task:

| File | When to read |
|------|-------------|
| [references/architecture.md](references/architecture.md) | Folder structure, domain layout, DI pattern |
| [references/patterns.md](references/patterns.md) | Code patterns: models, endpoints, services, repositories |
| [references/database.md](references/database.md) | Migrations, BaseTable CRUD methods, SQLModel conventions |
| [references/devops.md](references/devops.md) | Docker, Celery tasks, testing, env config, security, checklist |

## Quick Start: Adding a New Domain

1. Create `app/api/{domain}/` with `__init__.py`, `models.py`, `points.py`, `service.py`
2. Create `app/db/{entity}/` with `model.py` (SQLModel) and `table.py` (BaseTable)
3. Write migration in `db/migrations/{number}_{description}.sql`
4. Register router in `app/api/api.py`
5. Add tests in `tests/api/test_{domain}.py` and `tests/db/test_{entity}.py`

See [references/patterns.md](references/patterns.md) for complete code templates.
