# Layered FastAPI Pattern

Use this pattern when generating or refactoring FastAPI modules.

## Recommended Feature Layout

```text
app/
  main.py
  api/
    routes/
      users.py
  controllers/
    users_controller.py
  services/
    users_service.py
  schemas/
    users.py
  dependencies/
    auth.py
    db.py
  core/
    config.py
    constants.py
    exceptions.py
```

Exact folder names can vary, but the responsibility split should stay clear.

## Responsibility Matrix

### Route module

- owns URL definitions
- declares request params and `response_model`
- injects dependencies
- delegates to controller functions quickly

Example sketch:

```python
from fastapi import APIRouter, Depends, status

from app.controllers.users_controller import create_user_controller
from app.schemas.users import CreateUserRequest, UserResponseEnvelope

router = APIRouter(prefix="/users", tags=["users"])


@router.post(
    "/",
    response_model=UserResponseEnvelope,
    status_code=status.HTTP_201_CREATED,
)
async def create_user(payload: CreateUserRequest):
    return await create_user_controller(payload)
```

### Controller module

- accepts validated route inputs
- applies additional application checks
- orchestrates one or more services
- catches expected exceptions and maps them to HTTP-facing errors
- returns the final response schema

Example sketch:

```python
from fastapi import HTTPException, status

from app.schemas.common import ApiResponse
from app.schemas.users import CreateUserRequest, UserRead
from app.services.users_service import create_user


async def create_user_controller(payload: CreateUserRequest) -> ApiResponse[UserRead]:
    try:
        user = await create_user(payload)
        return ApiResponse[UserRead](
            status="success",
            message="User created successfully",
            data=UserRead.model_validate(user, from_attributes=True),
        )
    except ValueError as exc:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(exc),
        ) from exc
```

### Service module

- handles business rules and persistence interaction
- returns plain results, domain objects, or ORM entities
- does not return `JSONResponse`
- does not know about route decorators
- avoids broad `try/except` wrapping unless recovering or enriching a specific failure

Example sketch:

```python
async def create_user(payload: CreateUserRequest):
    existing_user = await users_repo.get_by_email(payload.email)
    if existing_user:
        raise ValueError("Email already exists")

    return await users_repo.create(payload.model_dump())
```

## Practical Notes

- If validation is purely structural, let FastAPI and Pydantic handle it at the route boundary.
- If validation is cross-field, business-aware, or depends on repository state, keep it in the controller or a domain helper.
- If the application already uses repositories beneath services, keep that extra layer; the key rule is still that HTTP concerns stop at the controller.
- If a service is reused by jobs, scripts, and events, that is a good sign that the layer boundary is healthy.
