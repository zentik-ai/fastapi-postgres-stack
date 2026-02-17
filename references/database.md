# Database Reference

## Migrations with dbmate

Migrations live in `db/migrations/` with format: `{number}_{description}.sql`

### Migration File Structure

```sql
-- migrate:up
CREATE TABLE IF NOT EXISTS entities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP,
    deleted_at TIMESTAMP
);

CREATE INDEX idx_entities_email ON entities(email);
CREATE INDEX idx_entities_status ON entities(status);

-- migrate:down
DROP TABLE IF EXISTS entities;
```

### dbmate Commands

```bash
dbmate up              # Apply migrations
dbmate new create_users_table  # Create new migration
dbmate down            # Rollback last migration
dbmate status          # View migration status
```

## BaseTable — Generic CRUD Methods

`BaseTable` (app/base/base_table.py) provides reusable async CRUD operations:

| Method | Description |
|--------|-------------|
| `first_or_default(id)` | Get by ID or None |
| `first_by(**kwargs)` | Get first matching filter |
| `all()` | Get all records |
| `all_by(**kwargs)` | Get all matching filters |
| `all_count(**kwargs)` | Count matching records |
| `insert_model(model)` | Insert new record |
| `update(id, data)` | Update by ID |
| `update_model(model)` | Update with full model |
| `delete(id)` | Hard delete by ID |
| `soft_delete(id)` | Soft delete (sets `deleted_at`) |

## Environment Configuration (app/config.py)

```python
import os
from dotenv import load_dotenv

load_dotenv()

# App
APP_NAME = os.getenv("APP_NAME", "my-api")
APP_ENVIRONMENT = os.getenv("APP_ENVIRONMENT", "local")
APP_VERSION = os.getenv("APP_VERSION", "1.0.0")

# Database
DB_HOST = os.getenv("DB_HOST", "localhost")
DB_PORT = os.getenv("DB_PORT", "5432")
DB_NAME = os.getenv("DB_NAME", "mydb")
DB_USERNAME = os.getenv("DB_USERNAME", "postgres")
DB_PASSWORD = os.getenv("DB_PASSWORD", "")

# Redis
REDIS_HOST = os.getenv("REDIS_HOST", "localhost")
REDIS_PORT = int(os.getenv("REDIS_PORT", 6379))
REDIS_PASSWORD = os.getenv("REDIS_PASSWORD", "")

# External Services
EXTERNAL_API_URL = os.getenv("EXTERNAL_API_URL", "http://localhost:8000")
EXTERNAL_API_TOKEN = os.getenv("EXTERNAL_API_TOKEN", "")
```

## .env Template

```ini
# ==================== APP ====================
APP_NAME=my-api
APP_ENVIRONMENT=local   # local, dev, staging, prod
APP_VERSION=1.0.0
LOG_LEVEL=DEBUG

# ==================== DATABASE ====================
DB_HOST=localhost
DB_PORT=5432
DB_NAME=mydb
DB_USERNAME=postgres
DB_PASSWORD=secret

# ==================== REDIS ====================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=secret

# ==================== SENTRY ====================
SENTRY_DSN=
SENTRY_ENVIRONMENT=local

# ==================== EXTERNAL SERVICES ====================
EXTERNAL_API_URL=http://localhost:8000
EXTERNAL_API_TOKEN=
```

## Environment-Aware Configuration

```python
if APP_ENVIRONMENT == "local":
    logger.add(sys.stderr, level="DEBUG")
else:
    sentry_sdk.init(dsn=SENTRY_DSN)
    logger.add(sink, serialize=True)
```
