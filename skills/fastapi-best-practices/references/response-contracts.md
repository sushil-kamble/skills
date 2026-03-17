# Response Contracts

Use consistent response schemas throughout the API.

## Default Envelope

Prefer a shared response model with:

- `status`
- `message`
- `data`

Example:

```python
from typing import Generic, TypeVar

from pydantic import BaseModel

T = TypeVar("T")


class ApiResponse(BaseModel, Generic[T]):
    status: str
    message: str
    data: T | None = None
```

## Success Patterns

Single resource:

```python
class UserRead(BaseModel):
    id: int
    email: str


class UserResponse(ApiResponse[UserRead]):
    pass
```

Collection response:

```python
class UserListData(BaseModel):
    items: list[UserRead]
    total: int


class UserListResponse(ApiResponse[UserListData]):
    pass
```

Action response with no payload:

```python
class EmptyResponse(ApiResponse[None]):
    pass
```

## Error Patterns

If the project uses an envelope for errors too, keep it consistent.

Example:

```python
class ErrorData(BaseModel):
    code: str | None = None
    details: dict | list | None = None


class ErrorResponse(ApiResponse[ErrorData]):
    pass
```

Guidelines:

- Keep `message` client-safe.
- Put machine-readable details in structured fields, not free-form strings only.
- Do not expose stack traces or raw SQL/driver errors.
- Make validation and domain errors look intentional and repeatable.

## FastAPI Integration

- Prefer `response_model=...` on route decorators.
- Use separate request and response schemas.
- Validate ORM/domain objects into response models with `model_validate(..., from_attributes=True)` when needed.
- Avoid returning raw dicts when a named schema would improve clarity.

## What to Avoid

- mixed response shapes across similar endpoints
- `data` sometimes being a dict, list, string, or boolean without a schema contract
- embedding HTTP-only metadata deep inside service return values
- building response payloads in services instead of controllers
