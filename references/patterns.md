# Code Patterns Reference

## Domain Structure

Each domain in `app/api/` follows this layout:

```
{domain}/
├── __init__.py
├── models.py       # Pydantic Request/Response models
├── points.py       # FastAPI Router with endpoints
└── service.py      # Business logic
```

## Pydantic Models (app/api/{domain}/models.py)

```python
from pydantic import BaseModel, Field
from typing import Optional

class CreateEntityRequest(BaseModel):
    name: str = Field(..., description="Entity name", min_length=1, max_length=100)
    email: str = Field(..., description="Email address")
    optional_field: Optional[str] = Field(None, description="Optional data")

class EntityResponse(BaseModel):
    success: bool = Field(..., description="Operation success status")
    data: Optional[dict] = Field(None, description="Response data")
    error: Optional[str] = Field(None, description="Error message if failed")
```

## Endpoints (app/api/{domain}/points.py)

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession

from app.db.database import get_db
from app.api.{domain}.models import CreateEntityRequest, EntityResponse
from app.api.{domain}.service import DomainService

router = APIRouter()

@router.post("/", response_model=EntityResponse,
    summary="Create a new entity",
    responses={201: {"description": "Entity created"}, 401: {"description": "Unauthorized"}}
)
async def create_entity(
    request: CreateEntityRequest,
    db: AsyncSession = Depends(get_db)
) -> EntityResponse:
    """
    Create a new entity.

    - **name**: Entity name (required, max 100 chars)
    - **email**: Email address (required)
    """
    service = DomainService(db)
    return await service.create(request)

@router.get("/{entity_id}")
async def get_entity(entity_id: str, db: AsyncSession = Depends(get_db)):
    try:
        service = DomainService(db)
        return await service.get_entity(entity_id)
    except EntityNotFoundError as e:
        raise HTTPException(status_code=404, detail=str(e))
```

## Service Pattern (app/api/{domain}/service.py)

```python
class DomainService:
    """Business logic for Domain"""

    def __init__(self, db_context: AsyncSession):
        self.entity_table = EntityTable(db_context)
        self.external_service = ExternalService()

    async def process_business_logic(self, data: RequestModel) -> ResponseModel:
        # 1. Validate business rules
        # 2. Interact with database
        # 3. Call external services if needed
        # 4. Return response
        pass

    async def get_entity(self, entity_id: str) -> EntityModel:
        entity = await self.entity_table.first_or_default(entity_id)
        if not entity:
            raise EntityNotFoundError(f"Entity {entity_id} not found")
        return entity
```

## Repository Pattern (app/db/{entity}/table.py)

```python
from app.base.base_table import BaseTable
from app.db.{entity}.model import EntityModel

class EntityTable(BaseTable):
    """Repository for Entity operations"""

    __tablename__: str = "entities"

    def __init__(self, db_context: AsyncSession):
        super().__init__(db_context, EntityModel)

    async def find_by_custom_field(self, value: str) -> Optional[EntityModel]:
        return await self.first_by(custom_field=value)
```

## SQLModel Entity (app/db/{entity}/model.py)

```python
import uuid as uuid_pkg
from datetime import datetime
from typing import Optional
from sqlmodel import Field, SQLModel

class EntityModel(SQLModel, table=True):
    """Database model for Entity"""

    __tablename__ = "entities"

    id: uuid_pkg.UUID = Field(
        default_factory=uuid_pkg.uuid4,
        primary_key=True,
        index=True,
        nullable=False,
    )
    name: str = Field(nullable=False)
    email: str = Field(nullable=False)
    status: str = Field(default="active")
    created_at: datetime = Field(default_factory=lambda: datetime.utcnow())
    updated_at: Optional[datetime] = Field(default=None)
    deleted_at: Optional[datetime] = Field(default=None)  # Soft delete
```

## Error Handling

```python
# Define domain-specific exceptions
class EntityNotFoundError(Exception):
    """Raised when entity is not found in database"""
    pass

class ValidationError(Exception):
    """Raised when validation fails"""
    pass
```

## Logging

```python
from loguru import logger

logger.info(f"Processing request for user {user_id}")
logger.debug(f"Full payload: {payload}")       # Dev only
logger.warning(f"Retry attempt {retry_count} for {operation}")
logger.error(f"Failed to process: {error}")
logger.exception(error)                         # Includes traceback
```

## Async Best Practices

```python
# Correct: use async/await consistently
async def process_items(items: List[Item]) -> List[Result]:
    results = []
    for item in items:
        result = await process_single_item(item)
        results.append(result)
    return results

# Correct: parallelize when possible
async def process_items_parallel(items: List[Item]) -> List[Result]:
    tasks = [process_single_item(item) for item in items]
    return await asyncio.gather(*tasks)

# WRONG: do NOT mix sync and async
def bad_function():
    result = asyncio.run(async_operation())  # Never do this in async context
```

## DB Transactions

```python
# Implicit transaction (auto-commit in BaseTable)
async def create_entity(self, data: CreateRequest) -> EntityModel:
    entity = EntityModel(**data.model_dump())
    return await self.entity_table.insert_model(entity)

# Explicit transaction (multiple operations)
async def complex_operation(self, data: ComplexRequest):
    try:
        entity = await self.entity_table.insert_model(entity_data)
        await self.related_table.insert_model(related_data)
        await self.db_context.commit()
        return entity
    except Exception as e:
        await self.db_context.rollback()
        raise e
```

## Auth Middleware Pattern

```python
# app/middleware/auth_middleware.py
class AuthMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, token: str, public_paths: list[str]):
        super().__init__(app)
        self.token = token
        self.public_paths = public_paths

    async def dispatch(self, request: Request, call_next):
        if request.method == "OPTIONS":
            return await call_next(request)
        if self._is_public_path(request.url.path):
            return await call_next(request)
        auth_header = request.headers.get("Authorization")
        if not auth_header or not self._validate_token(auth_header):
            return JSONResponse({"error": "Unauthorized"}, status_code=401)
        return await call_next(request)
```

## Security Headers

```python
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    return response
```

## Input Validation with Pydantic

```python
from pydantic import BaseModel, Field, validator, EmailStr

class UserCreateRequest(BaseModel):
    email: EmailStr  # Automatic email validation
    password: str = Field(..., min_length=8, max_length=128)

    @validator('password')
    def password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v
```
