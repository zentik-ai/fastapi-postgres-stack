# Architecture Reference

## Folder Structure

```
project-root/
├── app/                        # Main source code
│   ├── main.py                 # FastAPI application entry point
│   ├── config.py               # Centralized configuration
│   ├── celery_app.py           # Celery configuration
│   ├── celery_worker.py        # Celery worker
│   │
│   ├── api/                    # Presentation layer (HTTP endpoints)
│   │   ├── __init__.py
│   │   ├── api.py              # Main router grouping all routers
│   │   └── {domain}/           # Business domain module
│   │       ├── models.py       # Request/Response Pydantic models
│   │       ├── points.py       # Endpoints/Routes (FastAPI Router)
│   │       └── service.py      # Business logic
│   │
│   ├── base/                   # Reusable base classes
│   │   └── base_table.py       # Generic BaseTable for CRUD
│   │
│   ├── celery/                 # Async tasks
│   │   ├── tasks.py            # Celery task definitions
│   │   └── utils.py            # Task utilities
│   │
│   ├── core/                   # Core config (advanced settings)
│   │   └── config.py           # pydantic-settings configuration
│   │
│   ├── db/                     # Data layer
│   │   ├── database.py         # DB connection and sessions
│   │   ├── models.py           # Shared models (if applicable)
│   │   ├── generate_ddl.py     # Script to generate DDL
│   │   └── {entity}/           # DB module per entity
│   │       ├── model.py        # SQLModel/SQLAlchemy model
│   │       └── table.py        # Repository/Table class (CRUD)
│   │
│   ├── handler/                # Global exception handlers
│   │   └── exception_handlers.py
│   │
│   ├── middleware/             # FastAPI middlewares
│   │   ├── auth_middleware.py
│   │   ├── not_found_middleware.py
│   │   └── validation_error_middleware.py
│   │
│   ├── services/               # External service integrations
│   │   └── {service_name}/
│   │       ├── service.py      # Client/Adapter
│   │       ├── models.py       # Service-specific models
│   │       └── constants.py    # Service constants
│   │
│   └── utils/                  # Shared utilities
│       └── timezone.py
│
├── bruno/                      # API request collection (Bruno)
│   └── {project-name}/
│       ├── bruno.json
│       ├── collection.bru
│       ├── environments/
│       └── api/
│
├── db/                         # Database migrations
│   └── migrations/
│       ├── 001_initial.sql
│       └── ...
│
├── docker/                     # Auxiliary Dockerfiles
│   ├── db/Dockerfile
│   └── dbmate/Dockerfile
│
├── docs/                       # Project documentation (*.md)
├── media/                      # Media files (audio/, image/, video/)
├── tests/                      # Tests (mirrors app/)
│   ├── conftest.py
│   ├── api/
│   ├── db/
│   └── services/
│
├── .env                        # Environment variables (DO NOT commit)
├── .env.example                # Env variable template
├── .gitignore
├── .pre-commit-config.yaml
├── dbmate.yaml
├── docker-compose.yml
├── Dockerfile
├── entrypoint.sh
├── Makefile
├── poetry.lock
├── poetry.toml
├── pyproject.toml
└── README.md
```

## Middleware Order (app/main.py)

Middlewares must be added in this exact order:

```python
# 1. CORS (must be first)
app.add_middleware(CORSMiddleware, allow_origins=[...], ...)

# 2. Authentication
app.add_middleware(AuthMiddleware, token=API_TOKEN, public_paths=["/health", "/docs", "/openapi.json"])

# 3. Validation Errors
app.add_middleware(ValidationErrorMiddleware)

# 4. Not Found Handler
app.add_middleware(NotFoundMiddleware)
```

## Dependency Injection Pattern

DB sessions are injected via FastAPI's `Depends`:

```python
# app/db/database.py
async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

# In endpoints
@router.post("/")
async def create_item(
    request: RequestModel,
    db: AsyncSession = Depends(get_db)
):
    service = DomainService(db)
    return await service.create(request)
```
