# Quality Guidelines

> Code quality standards for backend development.

---

## Overview

Go backend with golangci-lint v2.7 + unit/integration test tags. All checks must pass before a PR is mergeable.

---

## CI Checks (must all pass)

```bash
# Unit tests
cd backend && go test -tags=unit ./...

# Integration tests
cd backend && go test -tags=integration ./...

# Lint
cd backend && golangci-lint run ./...
```

Go version: **1.25.7** (pinned in CI).

---

## Test Organization

Tests use build tags to separate unit from integration:

```go
//go:build unit
// +build unit

package service_test
```

```go
//go:build integration
// +build integration

package repository_test
```

- Unit tests: stub/mock all dependencies, no real DB
- Integration tests: real PostgreSQL + Redis connections

When adding a new interface method, update ALL test stubs that implement the interface:

```bash
# Find all stubs for an interface
grep -r "type.*Stub.*struct\|type.*Mock.*struct" backend/internal/
```

---

## Forbidden Patterns

- `fmt.Println` / `log.Println` in business code — use `logger` package
- `c.JSON(http.StatusXxx, ...)` directly in handlers — use `response.*` helpers
- Raw `zap.NewNop()` / `zap.NewExample()` outside tests
- Editing files in `backend/ent/` that are not in `ent/schema/` — they are generated
- Ignoring returned errors without explicit reason
- `panic()` outside of tests or startup code

---

## Required Patterns

- Every new handler must use `response.ErrorFrom(c, err)` for error responses
- Every new repository implementation must implement the interface defined in `internal/service/`
- Constructor functions return the interface type, not the concrete struct type
- Sentinel errors defined as `var ErrXxx = infraerrors.NotFound(...)` at service level
- Context (`ctx context.Context`) must be the first parameter in all DB-touching functions

---

## PR Checklist

- [ ] `go test -tags=unit ./...` passes
- [ ] `go test -tags=integration ./...` passes
- [ ] `golangci-lint run ./...` has no new issues
- [ ] If interface changed: all test stubs updated
- [ ] If Ent schema changed: `go generate ./ent` run and generated files committed
- [ ] No secrets or tokens in logs or responses
- [ ] New code uses structured logging via `logger` package
