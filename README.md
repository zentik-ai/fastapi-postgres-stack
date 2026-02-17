# FastAPI + PostgreSQL Stack

A production-ready architecture guide and template for building scalable API services using FastAPI and PostgreSQL. This project establishes best practices for domain-driven design, asynchronous database operations, and robust observability.

## 🚀 Overview

This repository serves as a blueprint for modern Python backend development. It emphasizes a layered architecture that separates presentation, business logic, and data access, ensuring maintainability and testability at scale.

## 🛠️ Tech Stack

### Core
- **Runtime**: Python 3.11+ with Poetry for dependency management.
- **Web Framework**: [FastAPI](https://fastapi.tiangolo.com/) for high-performance async API development.
- **Validation**: [Pydantic v2](https://docs.pydantic.dev/) for data modeling and settings management.

### Database & Persistence
- **Database**: PostgreSQL 15+.
- **ORM/Query Builder**: [SQLModel](https://sqlmodel.tiangolo.com/) (SQLAlchemy + Pydantic integration) with [asyncpg](https://github.com/MagicStack/asyncpg) driver.
- **Migrations**: [dbmate](https://github.com/amacneil/dbmate) for reliable database schema evolution.

### Infrastructure & Tasks
- **Cache/Queue**: Redis for caching and as a broker for [Celery](https://docs.celeryq.dev/).
- **Async Tasks**: Celery for background job processing.
- **Observability**: Sentry for error tracking, Loguru for structured logging, and OpenTelemetry for tracing.

### Testing & Quality
- **Testing**: [pytest](https://docs.pytest.org/) with `pytest-asyncio` and `pytest-cov`.
- **Quality**: pre-commit hooks, pylint, and isort.

## 🏗️ Architecture

The project follows a **Layered Architecture** pattern:

1.  **Presentation Layer (`app/api/`)**: FastAPI Routers handling HTTP requests and Pydantic validation.
2.  **Business Layer (`app/api/{domain}/service.py`)**: Core business logic and orchestration.
3.  **Data Access Layer (`app/db/`)**: CRUD operations and database abstractions using the Repository pattern.
4.  **Database Layer**: PostgreSQL managed via SQLModel/SQLAlchemy.

### Folder Structure Highlights
```text
project-root/
├── app/                  # Main source code
│   ├── api/              # Presentation layer (HTTP endpoints)
│   ├── db/               # Data layer (Repositories & Models)
│   ├── base/             # Reusable base classes
│   ├── middleware/       # FastAPI middlewares
│   └── services/         # External service integrations
├── db/                   # Database migrations
├── tests/                # Comprehensive test suite
└── bruno/                # API request collection
```

## 📖 Key Conventions

- **Singular Domains**: Use singular snake_case for domain modules (e.g., `user/`, `order/`).
- **Plural Tables**: Database tables should be plural snake_case (e.g., `users`, `orders`).
- **Dependency Injection**: Database sessions are injected into endpoints via FastAPI's `Depends`.
- **Async First**: All I/O operations (DB, HTTP clients, Cache) are asynchronous.

## 🚦 Quick Start

### Adding a New Domain
1.  Define models in `app/api/{domain}/models.py`.
2.  Implement business logic in `app/api/{domain}/service.py`.
3.  Create API routes in `app/api/{domain}/points.py`.
4.  Define the database table in `app/db/{entity}/model.py` and repository in `table.py`.
5.  Create a migration: `dbmate new <description>`.
6.  Register the router in `app/api/api.py`.

## 🛠️ Development & Tooling

The project includes several tools to streamline development and deployment:

- **Makefile**: Common tasks like `install`, `run`, `test`, `lint`, and `format` are automated via the `Makefile`.
- **Docker & Compose**: Pre-configured `Dockerfile` and `docker-compose.yml` for local development, including PostgreSQL, Redis, and dbmate.
- **Bruno**: An API request collection is provided in the `bruno/` directory for testing endpoints.
- **Pre-commit**: Automated linting and formatting hooks to maintain code quality.

## ✅ New Project Checklist

Starting a new service with this stack? Follow the [New Project Checklist](references/devops.md#new-project-checklist) to ensure everything from environment variables to production monitoring is correctly configured.

## 📚 Documentation & References

For more detailed information, please refer to the following:
- [Architecture & Structure](references/architecture.md)
- [Database & Migrations](references/database.md)
- [Design Patterns](references/patterns.md)
- [DevOps & Security](references/devops.md)
- [Full Tech Stack Versions](references/stack.md)
