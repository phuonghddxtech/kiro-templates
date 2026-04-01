# Python Stack

## When to Use Python vs Node/Express

| Tiêu chí | Python (FastAPI) | Node (Express) |
|----------|-----------------|----------------|
| **Team** | Python/ML team | JS/TS team |
| **Data/ML** | ✅ Native ecosystem | Requires bridges |
| **API dev** | ✅ FastAPI auto-docs | Manual Swagger |
| **Real-time** | Less common | ✅ Socket.io native |
| **Scripting/automation** | ✅ Preferred | Less common |
| **Microservices** | ✅ Supported | ✅ Preferred |

## Approved Stack

| Layer | Choice |
|-------|--------|
| **Runtime** | Python 3.11+ |
| **Framework** | FastAPI (REST) |
| **Validation** | Pydantic v2 |
| **ORM** | SQLAlchemy 2.0 (async) |
| **Migrations** | Alembic |
| **Database** | PostgreSQL |
| **Cache** | Redis (aioredis) |
| **Queue** | Celery + Redis |
| **Auth** | python-jose (JWT) + passlib (bcrypt) |
| **Logging** | structlog (structured JSON) |
| **Testing** | pytest + pytest-asyncio + httpx |
| **Linting** | ruff + mypy |
| **Package mgr** | uv |

## Project Structure
```
src/
├── api/v1/
│   ├── routes/          # Route definitions (thin)
│   └── dependencies.py  # FastAPI dependencies
├── core/
│   ├── config.py        # pydantic-settings
│   ├── security.py      # JWT, bcrypt
│   └── exceptions.py    # AppError
├── models/              # SQLAlchemy models
├── schemas/             # Pydantic request/response
├── services/            # Business logic
├── repositories/        # Data access
├── workers/             # Celery tasks
└── main.py
```

## Architecture
```
Route → Dependency → Service → Repository → Database
```

## Quick Setup
```bash
uv init my-app && cd my-app
uv add fastapi uvicorn sqlalchemy asyncpg alembic pydantic-settings \
       python-jose passlib[bcrypt] structlog celery redis
uv add --dev pytest pytest-asyncio httpx ruff mypy
```

## Config (pydantic-settings)
```python
# src/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    redis_url: str
    jwt_secret: str
    jwt_expires_minutes: int = 15

    class Config:
        env_file = ".env"

settings = Settings()
```

## Custom Exception
```python
# src/core/exceptions.py
from fastapi import HTTPException

class AppError(HTTPException):
    def __init__(self, message: str, status_code: int = 500, code: str = "INTERNAL_ERROR"):
        super().__init__(status_code=status_code, detail={"code": code, "message": message})
```

## Response Envelope
```python
# ✅ Always wrap responses — consistent with other services
{"success": True, "data": user}
{"success": True, "data": users, "pagination": {"page": 1, "limit": 20, "total": 100}}
{"success": False, "error": {"code": "VALIDATION_ERROR", "message": "..."}}
```

## Naming Conventions
```
Files/folders:   snake_case   (user_service.py)
Classes:         PascalCase   (UserService)
Functions/vars:  snake_case   (find_by_id)
Constants:       UPPER_SNAKE  (MAX_RETRIES)
```

## Key Rules
- **Type hints on every function** — enforced by mypy
- Use `AppError` for all operational errors — never raise raw `Exception`
- All DB calls inside repositories — never in routes or services directly
- Use `pydantic-settings` for config — never `os.environ` directly
- Celery tasks must have `max_retries` and `default_retry_delay`
- `ruff check` and `mypy` must pass before PR
