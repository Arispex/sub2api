# Error Handling

> How errors are handled in this project.

---

## Overview

All errors use `ApplicationError` from `internal/pkg/errors`. Handlers convert errors to HTTP responses via `response.ErrorFrom()`. Business logic never writes HTTP status codes directly.

---

## Error Types

The central type is `ApplicationError` (alias `Error`) in `internal/pkg/errors/errors.go`:

```go
type ApplicationError struct {
    Status  // Code (HTTP status int32), Reason (machine-readable string), Message (human-readable)
    cause error
}
```

Constructor shortcuts:

```go
infraerrors.NotFound("ACCOUNT_NOT_FOUND", "account not found")
infraerrors.BadRequest("ACCOUNT_NIL_INPUT", "account input cannot be nil")
infraerrors.Unauthorized("...", "...")
infraerrors.Forbidden("...", "...")
infraerrors.Internal("...", "...")
infraerrors.Conflict("...", "...")
```

---

## Sentinel Errors

Define sentinel errors at the service level, not in handlers or repositories:

```go
// internal/service/account_service.go
var (
    ErrAccountNotFound = infraerrors.NotFound("ACCOUNT_NOT_FOUND", "account not found")
    ErrAccountNilInput = infraerrors.BadRequest("ACCOUNT_NIL_INPUT", "account input cannot be nil")
)
```

---

## Error Propagation

- Repository returns `*ApplicationError` or wraps DB errors: `infraerrors.FromError(err)` or `.WithCause(err)`
- Service checks sentinel errors with `errors.Is()` and may wrap/re-return
- Handler calls `response.ErrorFrom(c, err)` to translate to HTTP response — **no manual status code selection in handlers**

```go
// handler
result, err := h.service.GetByID(c.Request.Context(), id)
if err != nil {
    response.ErrorFrom(c, err)
    return
}
```

---

## API Error Responses

Standard JSON envelope:

```json
{
  "code": 404,
  "message": "account not found",
  "reason": "ACCOUNT_NOT_FOUND",
  "metadata": {}
}
```

`response` package helpers:

```go
response.Success(c, data)           // 200
response.Created(c, data)           // 201
response.Accepted(c, data)          // 202
response.Paginated(c, items, ...)   // 200 paginated
response.ErrorFrom(c, err)          // auto-maps ApplicationError → HTTP status
response.Unauthorized(c, msg)       // 401
response.BadRequest(c, msg)         // 400
```

Backend `success` responses must NOT include user-facing message strings — the frontend generates display text from status codes and `error.message`.

---

## Common Mistakes

- **Returning raw `error` from service to handler** — always wrap in `ApplicationError` so the status code is correct
- **Writing `c.JSON(http.StatusXxx, ...)` directly in handlers** — use `response.*` helpers instead
- **Hiding error details** — `error.message` should be the actual cause, not a generic string
- **Returning localized/translated messages from backend** — backend returns raw reasons; frontend generates display text
