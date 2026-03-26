# Directory Structure

> How backend code is organized in this project.

---

## Overview

Go backend using Gin (HTTP) + Ent ORM + PostgreSQL + Redis. Code follows a layered architecture: handler → service → repository.

---

## Directory Layout

```
backend/
├── cmd/
│   └── server/          # Main entry point (main.go)
├── ent/
│   ├── schema/          # Ent entity schema definitions (source of truth)
│   └── ...              # Generated Ent code (DO NOT edit manually)
├── internal/
│   ├── handler/         # HTTP handlers (Gin) - request/response only
│   │   ├── admin/       # Admin-only handlers
│   │   ├── dto/         # Data transfer objects (handler ↔ service)
│   │   └── *.go
│   ├── service/         # Business logic + domain models + repository interfaces
│   │   └── *.go
│   ├── repository/      # DB implementations of service interfaces
│   │   └── *.go
│   ├── server/          # Gin router setup, middleware
│   │   └── middleware/
│   ├── pkg/             # Shared internal packages (not domain-specific)
│   │   ├── errors/      # ApplicationError type
│   │   ├── logger/      # Unified zap logger
│   │   ├── pagination/  # Pagination helpers
│   │   └── response/    # Gin HTTP response helpers
│   └── domain/          # Pure domain constants/value objects (no deps)
├── migrations/          # Raw SQL migration files
├── resources/           # Embedded static resources
└── go.mod
```

---

## Module Organization

Each feature typically spans three files following the naming pattern `{feature}_{responsibility}.go`:

- `internal/handler/{feature}_handler.go` - HTTP handler struct + route methods
- `internal/service/{feature}_service.go` - Service struct + domain model + repository interface
- `internal/repository/{feature}_repo.go` - Repository implementation

Large features are split into multiple files by sub-concern, e.g.:
- `account_service.go`, `account_expiry_service.go`, `account_usage_service.go`
- `gateway_handler.go`, `gateway_handler_chat_completions.go`, `gateway_handler_responses.go`

---

## Naming Conventions

- Files: `snake_case.go`
- Packages: single lowercase word matching directory name
- Types/structs: `PascalCase`
- Interfaces: defined in `service/` package, named after what they do (e.g., `AccountRepository`, `SchedulerCache`)
- Constructors: `New{TypeName}(...)` returning interface type
- Sentinel errors: `var ErrXxx = infraerrors.NotFound(...)` at package level in service file
- Test files: `{feature}_test.go` with build tag `//go:build unit` or `//go:build integration`

---

## Examples

- Handler: `backend/internal/handler/api_key_handler.go`
- Service + interface: `backend/internal/service/account_service.go`
- Repository impl: `backend/internal/repository/account_repo.go`
- Schema: `backend/ent/schema/account.go`
