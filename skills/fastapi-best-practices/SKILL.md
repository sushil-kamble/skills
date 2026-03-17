---
name: fastapi-best-practices
description: Use this skill whenever the user is building, refactoring, reviewing, or debugging a FastAPI backend or API module. Apply it for endpoint design, router organization, Pydantic schemas, controller/service layering, error handling, response contracts, dependency injection, and production-ready FastAPI project structure even if the user only loosely asks for a FastAPI API, backend cleanup, or best-practices review.
---

# FastAPI Best Practices

Use this skill to keep FastAPI codebases modular, predictable, and easy to test.

## Outcome

Produce FastAPI code that:

- follows a `routes -> controllers -> services` flow
- keeps routes thin and declarative
- validates inputs and shapes responses with Pydantic models
- centralizes request orchestration and exception handling in controllers
- keeps services focused on business logic and data access only
- uses FastAPI features the way the framework expects: `APIRouter`, dependencies, `HTTPException`, and `response_model`

## Architecture Rules

Use this structure unless the repository already has a stronger local convention that the user explicitly wants preserved:

1. `routes`
2. `controllers`
3. `services`

### Routes

Routes are transport wiring only.

- Define endpoints with `APIRouter`.
- Attach `prefix`, `tags`, dependencies, and `response_model` metadata at the router or endpoint level.
- Parse path/query/header/cookie/body inputs using FastAPI and Pydantic types.
- Delegate immediately to a controller.
- Do not place database logic, orchestration, or complex branching in route functions.

### Controllers

Controllers coordinate the request lifecycle.

- Treat the controller as the layer responsible for request-to-response orchestration.
- Validate or normalize request data that needs application-level checks beyond the route signature.
- If the project uses Pydantic request models, controllers may perform additional `model_validate(...)` or cross-field validation before calling services.
- Handle `try/except` here when translating domain/infrastructure failures into API responses.
- Raise or map `HTTPException` here when the goal is to send a client-facing HTTP error.
- Call one or more services as needed.
- Build and return the final response payload here using response Pydantic models.

### Services

Services contain business logic and persistence interaction.

- Keep services framework-light.
- Let services interact with the DB, repositories, external APIs, or other domain helpers.
- Return values or domain objects; do not build HTTP responses here.
- Do not add `try/except` blocks just to wrap and rethrow everything. Let meaningful exceptions bubble to controllers unless there is a real recovery step.
- Do not hard-code reusable constants, magic strings, status text, or environment-specific values.
- Do not depend on FastAPI request/response objects unless there is no reasonable alternative.

## Validation and Schemas

Prefer Pydantic models for request and response contracts.

- Create explicit request schemas, response schemas, and shared nested schemas.
- Keep input and output models separate when fields differ.
- Use Pydantic v2 style when the codebase supports it: `model_validate`, `model_dump`, `ConfigDict`, `field_validator`, and `model_validator`.
- Prefer `extra="forbid"` for strict request bodies unless the API intentionally allows unknown keys.
- Use `from_attributes=True` only when validating ORM/domain objects into response models.
- Keep business logic out of schema validators; validators should enforce data shape and local invariants, not orchestrate services.

If the repository says "pidentic" or similar, interpret that as `pydantic` unless the repo actually defines something custom.

## Response Contract

Prefer a consistent API envelope across the project.

The default response shape should be represented by Pydantic models and include:

- `status`
- `message`
- `data`

Use a typed response envelope when possible.

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

Guidance:

- Use `response_model` on endpoints so FastAPI validates, documents, and filters output.
- Keep `status` values consistent project-wide, such as `success` and `error`.
- Keep `message` human-readable and stable enough for clients and logs.
- Use domain-specific `data` models instead of raw dicts whenever possible.
- For empty-success cases, set `data=None` with a meaningful message.
- For paginated responses, wrap pagination metadata in a dedicated Pydantic model instead of overloading the envelope ad hoc.

## Error Handling

Follow this error policy:

- Use FastAPI's built-in validation for transport-level parsing whenever possible.
- Let controllers translate expected failures into consistent HTTP responses.
- Keep internal exception details out of client payloads.
- Use app-level exception handlers for cross-cutting patterns only when the repository benefits from a single global format.
- Prefer one consistent error body pattern throughout the API.

Good default split:

- route: wiring
- controller: validate, orchestrate, catch, map, respond
- service: execute business/data logic and return values

## FastAPI-Specific Practices

- Organize features with `APIRouter` instead of a monolithic `main.py`.
- Include routers from the application entrypoint.
- Use dependencies for auth, DB session acquisition, shared guards, and reusable request context.
- Annotate return types or use `response_model` explicitly.
- Prefer async only for truly async I/O paths; keep sync code sync.
- Avoid leaking ORM models directly from endpoints.
- Keep OpenAPI docs accurate through tags, response models, status codes, and summaries.

## How to Apply This Skill

When editing or generating FastAPI code:

1. Identify the current layering and whether the repo already has routes, controllers, and services.
2. Preserve existing conventions where reasonable, but steer new code toward `routes -> controllers -> services`.
3. Move validation, response shaping, and exception mapping into controllers.
4. Move DB and business operations into services.
5. Add or refine Pydantic request/response models.
6. Ensure endpoint decorators use accurate `response_model` declarations.
7. Standardize response bodies around `status`, `message`, and `data`.
8. Keep services free from unnecessary `try/except` wrappers.

## Review Checklist

Before finishing, check whether the code:

- keeps routes thin
- keeps controllers responsible for validation, orchestration, and response shaping
- keeps services free of HTTP concerns and broad exception wrapping
- avoids hard-coded constants where configuration or shared constants belong
- uses Pydantic models for request and response payloads
- returns a consistent response envelope
- uses FastAPI router organization and dependency injection cleanly

## Reference Files

- Read `references/layered-fastapi-pattern.md` when you need a concrete mental model for route/controller/service responsibilities.
- Read `references/response-contracts.md` when you need response-envelope guidance and examples.
