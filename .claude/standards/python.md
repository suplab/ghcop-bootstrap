# Python Coding Standards

> Mandatory standards for all Python code. Enforced by code review (`code-reviewer`, `python-developer`) and CI gates.

---

## Language & Runtime

- **Python 3.11+** minimum; target Python 3.12 for new projects
- Type annotations on all public functions, methods, and class attributes (enforced by `mypy --strict`)
- Use `pyproject.toml` for project configuration; no `setup.py` in new projects
- Manage dependencies with `uv` (preferred) or `pip-tools`; pin all transitive dependencies in `requirements.lock`

---

## Code Style

- Format with **Ruff** (`ruff format .`) — replaces Black, isort, and flake8
- Lint with **Ruff** (`ruff check .`) — all rules active; disable inline with `# noqa: RuleCode` + mandatory comment explaining why
- Maximum line length: 100 characters
- Use double quotes for strings
- Never `import *` from any module

```python
# CORRECT
from collections.abc import Sequence
from typing import Optional

def find_orders(customer_id: str, statuses: Sequence[str]) -> list[dict]:
    ...

# WRONG — wildcard import hides what's available
from mymodule import *
```

---

## Logging

- **Standard `logging` module only** — never `print()` in production code
- Use module-level loggers: `logger = logging.getLogger(__name__)`
- Use `%s` format strings (lazy evaluation) — never f-strings inside log calls
- Never log PII (names, emails, national IDs) at any level

```python
# CORRECT
import logging
logger = logging.getLogger(__name__)
logger.info("Order placed: order_id=%s customer_id=%s", order.id, order.customer_id)

# WRONG
print(f"Order placed: {order.id}")                   # print in production
logger.info(f"Order placed: {order.id}")             # f-string (evaluates eagerly)
```

---

## Error Handling

- Never use bare `except:` — always catch a specific exception class
- Never swallow exceptions silently — at minimum `logger.warning` or `logger.error` with `exc_info=True`
- Use custom domain exception classes inheriting from a project `BaseError`
- Use context managers (`with`) for all resource management (files, DB connections, HTTP sessions)

```python
# CORRECT
try:
    response = client.get_order(order_id)
except OrderNotFoundError:
    logger.warning("Order not found: order_id=%s", order_id)
    raise
except httpx.TimeoutException as exc:
    logger.error("Timeout fetching order: order_id=%s", order_id, exc_info=True)
    raise ServiceUnavailableError("Order service timed out") from exc

# WRONG
try:
    response = client.get_order(order_id)
except:          # bare except — catches SystemExit, KeyboardInterrupt
    pass         # silently swallowed
```

---

## Type Annotations

- All function signatures must be fully annotated (parameters and return type)
- Use `X | None` union syntax (Python 3.10+) instead of `Optional[X]`
- Use `collections.abc` types for parameters: `Sequence`, `Mapping`, `Iterable`; use concrete types only for return values
- Use `TypeVar` and `Generic` for reusable type-safe abstractions
- Use `@dataclass(frozen=True)` or Pydantic `BaseModel` for structured data — never `dict` for domain objects

```python
# CORRECT
from collections.abc import Sequence
from dataclasses import dataclass

@dataclass(frozen=True)
class OrderId:
    value: str

def process_orders(ids: Sequence[OrderId]) -> list[str]:
    return [id.value for id in ids]

# WRONG — untyped, mutable dict as domain object
def process_orders(ids):
    return [id["value"] for id in ids]
```

---

## Dependency Injection

- Use constructor injection for all dependencies
- Never use module-level global state for dependencies (no `db = create_engine(...)` at module top level)
- Use dependency injection frameworks (FastAPI `Depends`, `python-dependency-injector`) — not service locators

---

## Testing

- **pytest** with `pytest-asyncio` for async tests
- Minimum coverage: 80% line, 70% branch (enforced by `pytest-cov`)
- Unit tests: no I/O, no network, stub all collaborators with `unittest.mock.patch` or `pytest-mock`
- Integration tests: use Docker containers via `testcontainers-python`
- Never use `time.sleep()` in tests — use `asyncio.wait_for` or polling with `tenacity`

```python
# CORRECT
import pytest
from unittest.mock import AsyncMock

@pytest.mark.asyncio
async def test_place_order_publishes_event(order_service, mock_event_bus):
    mock_event_bus.publish = AsyncMock()
    await order_service.place_order(order_dto)
    mock_event_bus.publish.assert_called_once()

# WRONG
def test_place_order():
    import time
    time.sleep(2)  # never
```

---

## Project Structure

```
src/
  myapp/
    domain/           # Pure domain: entities, value objects, domain services
    application/      # Use cases, command/query handlers
    infrastructure/   # DB adapters, HTTP clients, message producers
    web/              # FastAPI routers, request/response models
    config/           # Settings, DI container wiring
tests/
  unit/
  integration/
pyproject.toml
```

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| `print()` in production | `logging.getLogger(__name__).info(...)` |
| Bare `except:` | `except SpecificError:` |
| `import *` | Explicit imports |
| Mutable default args (`def f(x=[])`) | `def f(x=None): if x is None: x = []` |
| `dict` for domain objects | `@dataclass(frozen=True)` or Pydantic `BaseModel` |
| `os.system()` for shell calls | `subprocess.run([...], check=True)` |
| Module-level DB connections | Injected via constructor / `Depends` |
| Global state in tests | Fixtures with `scope="function"` |

---

## Enforcement

- Pre-commit: `ruff format --check`, `ruff check`, `mypy`
- CI gate: `pytest --cov --cov-fail-under=80`
- Code review: `python-developer` and `code-reviewer` agents enforce these rules
