# FastAPI Engineering Standard

> Standards for all FastAPI applications. Complements `python.md`. Enforced by `python-developer` agent and CI gates.

---

## Project Layout

```
src/myapp/
  web/
    routers/          # One router per resource domain (orders.py, customers.py)
    models/           # Pydantic request/response schemas (separate from domain)
    dependencies.py   # Shared FastAPI Depends factories
  application/        # Use cases / command handlers (no FastAPI imports here)
  domain/             # Pure domain entities and value objects
  infrastructure/     # DB, HTTP clients, messaging
  config/
    settings.py       # Pydantic BaseSettings
    container.py      # Dependency injection wiring
main.py               # App factory — create_app() function
```

---

## Router Organisation

- One `APIRouter` per resource domain; include all routers in `main.py`
- Tag every router: `router = APIRouter(prefix="/v1/orders", tags=["Orders"])`
- All routes use explicit response models — never return raw dicts

```python
# CORRECT
from fastapi import APIRouter, Depends, status
from myapp.web.models.orders import OrderResponse, CreateOrderRequest
from myapp.web.dependencies import get_order_service

router = APIRouter(prefix="/v1/orders", tags=["Orders"])

@router.post(
    "/",
    response_model=OrderResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Place a new order",
)
async def create_order(
    body: CreateOrderRequest,
    service: OrderService = Depends(get_order_service),
) -> OrderResponse:
    result = await service.place_order(body.to_command())
    return OrderResponse.from_domain(result)

# WRONG — no response_model, returns dict, no status code
@router.post("/orders")
async def create_order(body: dict):
    return {"status": "ok"}
```

---

## Pydantic Schemas

- Separate Pydantic models for request input and response output — never expose domain entities directly
- Use `model_config = ConfigDict(from_attributes=True)` for ORM-to-schema mapping
- Field descriptions in every schema (they appear in OpenAPI docs)
- Use `model_validator` for cross-field validation; use `field_validator` for single-field rules
- All datetime fields use `datetime` with explicit UTC: `AwareDatetime` (Pydantic v2)

```python
from datetime import datetime
from pydantic import BaseModel, ConfigDict, Field, field_validator
from pydantic import AwareDatetime

class CreateOrderRequest(BaseModel):
    customer_id: str = Field(..., description="UUID of the customer placing the order")
    items: list[OrderItemRequest] = Field(..., min_length=1, description="Order line items")

    @field_validator("customer_id")
    @classmethod
    def validate_uuid(cls, v: str) -> str:
        import uuid
        uuid.UUID(v)  # raises ValueError if invalid
        return v

class OrderResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    order_id: str
    status: str
    created_at: AwareDatetime
```

---

## Dependency Injection

- All service/repository dependencies declared via `Depends` — never instantiated inside route functions
- Use `Annotated` type aliases to avoid repeating `Depends` at every route

```python
from typing import Annotated
from fastapi import Depends

# Declare once
OrderServiceDep = Annotated[OrderService, Depends(get_order_service)]
DatabaseDep = Annotated[AsyncSession, Depends(get_db_session)]

# Use everywhere
@router.get("/{order_id}", response_model=OrderResponse)
async def get_order(order_id: str, service: OrderServiceDep, db: DatabaseDep) -> OrderResponse:
    ...
```

---

## Error Handling

- Register a global exception handler for domain exceptions — never handle domain errors in route functions
- Return RFC 7807 `ProblemDetail`-compatible JSON for all errors
- Use `HTTPException` only for HTTP protocol errors (auth, rate limits) — not for domain errors

```python
# In main.py / app factory
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(OrderNotFoundError)
async def order_not_found_handler(request: Request, exc: OrderNotFoundError) -> JSONResponse:
    return JSONResponse(
        status_code=404,
        content={
            "type": "https://api.example.com/errors/order-not-found",
            "title": "Order Not Found",
            "status": 404,
            "detail": str(exc),
            "instance": str(request.url),
        },
    )
```

---

## Async / Await

- All route handlers must be `async def`
- All I/O operations (DB, HTTP, file) must be awaited
- Use `asyncio.gather()` for concurrent independent I/O operations
- Never call blocking I/O inside an `async def` — use `asyncio.run_in_executor` or `anyio.to_thread.run_sync`

```python
# CORRECT — concurrent async calls
async def enrich_order(order_id: str) -> EnrichedOrder:
    order, customer = await asyncio.gather(
        order_repo.find(order_id),
        customer_client.get(order.customer_id),
    )
    return EnrichedOrder(order=order, customer=customer)

# WRONG — blocking call inside async
async def enrich_order(order_id: str) -> EnrichedOrder:
    import requests
    response = requests.get(f"/customers/{order.customer_id}")  # blocks event loop
```

---

## Settings

- All configuration via `pydantic_settings.BaseSettings` — never hardcoded values
- Secrets loaded from environment variables or AWS Secrets Manager — never from files committed to source control
- Validate settings at startup — fail fast rather than at first use

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    database_url: str
    secret_key: str
    jwt_algorithm: str = "RS256"
    token_expire_minutes: int = 30

settings = AppSettings()  # raises ValidationError at import if required envs missing
```

---

## OpenAPI Documentation

- Set `title`, `description`, `version`, and `contact` in app factory
- Use `openapi_tags` to group endpoints with descriptions
- Every endpoint has `summary`, `description`, and documents all response codes
- Do not disable OpenAPI in production — protect it with auth if sensitive

---

## Middleware

- Register middleware in this order (outermost first): CORS, RequestID, Authentication, Logging, Compression
- Never put business logic in middleware
- Use `starlette.middleware.trustedhost.TrustedHostMiddleware` in production

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| Business logic in route functions | Move to application service / use case |
| Raw `dict` return from routes | `response_model=` with Pydantic schema |
| `HTTPException` for domain errors | Global exception handler returning ProblemDetail |
| Synchronous DB calls in async routes | `await` with async session (SQLAlchemy async) |
| Hardcoded config values | `BaseSettings` from env vars |
| `global` state for DB session | `Depends(get_db_session)` per request |
| Catch-all `except Exception` in routes | Specific exception handlers registered on app |
