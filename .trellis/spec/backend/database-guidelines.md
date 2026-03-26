# Database Guidelines

> Database patterns and conventions for this project.

---

## Overview

- ORM: **Ent** (entgo.io) — type-safe, code-generated
- DB: PostgreSQL 16
- Cache: Redis
- Schema source of truth: `backend/ent/schema/*.go`
- Generated code lives in `backend/ent/` — never edit it manually

---

## Schema Definitions

Each entity is a file in `backend/ent/schema/`. Every schema must include:

```go
// account.go
func (Account) Mixin() []ent.Mixin {
    return []ent.Mixin{
        mixins.TimeMixin{},        // created_at, updated_at (auto-managed)
        mixins.SoftDeleteMixin{},  // deleted_at (soft delete)
    }
}

func (Account) Annotations() []schema.Annotation {
    return []schema.Annotation{
        entsql.Annotation{Table: "accounts"}, // explicit table name
    }
}
```

After any schema change, regenerate:

```bash
cd backend
go generate ./ent
git add ent/   # commit generated code too
```

---

## Repository Pattern

Repository interfaces are defined in `internal/service/` (alongside domain models).
Implementations are in `internal/repository/`.

```go
// service/account_service.go — interface
type AccountRepository interface {
    Create(ctx context.Context, account *Account) error
    GetByID(ctx context.Context, id int64) (*Account, error)
    // ...
}

// repository/account_repo.go — implementation
type accountRepository struct {
    client *dbent.Client
    sql    sqlExecutor // for complex raw SQL
}

func NewAccountRepository(client *dbent.Client, sqlDB *sql.DB, ...) service.AccountRepository {
    return &accountRepository{...}
}
```

---

## Query Patterns

**Use Ent for standard CRUD:**

```go
// Create
r.client.Account.Create().
    SetName(account.Name).
    SetPlatform(account.Platform).
    Save(ctx)

// Query with filters
r.client.Account.Query().
    Where(dbaccount.Platform(platform)).
    Where(dbaccount.DeletedAtIsNil()). // soft delete filter
    All(ctx)
```

**Use raw SQL for complex operations (batch updates, aggregates):**

```go
_, err = r.sql.ExecContext(ctx,
    `UPDATE accounts SET last_used_at = $1 WHERE id = $2`,
    t, id,
)
```

---

## Soft Deletes

All entities use `SoftDeleteMixin`. Queries must filter `deleted_at IS NULL`.
Ent hooks handle this automatically for Ent queries; raw SQL must add the filter manually.

---

## Migrations

- Migration files: `backend/migrations/` (raw SQL)
- Naming: sequential numbering or timestamp prefix
- Never modify a migration after it has been applied to production

---

## Naming Conventions

- Tables: `snake_case` plural (`accounts`, `api_keys`, `account_groups`)
- Columns: `snake_case`
- Indexes: `{table}_{column(s)}_idx`
- JSON fields stored in `extra` column as `jsonb` for flexible per-platform metadata

---

## Common Mistakes

- **Forgetting to regenerate Ent after schema change** — code won't reflect schema edits until `go generate ./ent` is run
- **Not committing generated code** — CI breaks because `ent/` is out of sync
- **Editing generated files** — changes are overwritten on next `go generate`
- **Missing soft delete filter in raw SQL** — accidentally returns deleted records
- **Adding new interface method without updating test stubs** — compile error; search for all structs implementing the interface
