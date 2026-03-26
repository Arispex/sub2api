# Logging Guidelines

> How logging is done in this project.

---

## Overview

Logging uses **zap** (go.uber.org/zap) via a unified wrapper at `internal/pkg/logger/`.
Always use the logger package — never call `fmt.Println`, `log.Println`, or raw `zap` directly in business code.

---

## Logger API

```go
// Get global logger
logger.L() *zap.Logger

// Get request-scoped logger (preferred in handlers/services that have ctx)
logger.FromContext(ctx) *zap.Logger

// Legacy printf-style (for gradual migration only, prefer structured)
logger.LegacyPrintf(component, format, args...)
```

---

## Log Levels

| Level | When to use |
|-------|------------|
| `Debug` | Verbose internal state, disabled in production |
| `Info` | Normal lifecycle events: service start, successful key operations |
| `Warn` | Recoverable issues, fallbacks, retries, rate limits, backoffs |
| `Error` | Unrecoverable failures, unexpected conditions |
| `Fatal` | Startup failures only (terminates process) |

Use `Warn`, not `Warning`. Use `Error`, not `Err`.

---

## Structured Logging

Log with structured fields, not interpolated strings:

```go
// Correct
logger.FromContext(ctx).Warn("gateway.failover_switch_account",
    zap.Int64("account_id", accountID),
    zap.Int("upstream_status", failoverErr.StatusCode),
    zap.Int("switch_count", s.SwitchCount),
    zap.Int("max_switches", s.MaxSwitches),
)

// Wrong — hard to grep, hard to parse
logger.L().Warn(fmt.Sprintf("switching account %d, status=%d", accountID, status))
```

Message format: `{component}.{event}` — e.g., `gateway.failover_switch_account`, `account.quota_reset`.

---

## What to Log

- Key lifecycle events: request received/completed, account scheduled/unscheduled
- State transitions: account paused, rate-limited, unscheduled
- External dependency calls that fail or are slow
- Fallback/retry decisions with reason fields
- Business rule triggers: quota exceeded, session window closed, warmup skipped

---

## What NOT to Log

- API keys, tokens, passwords, secrets — ever
- Full request/response bodies (may contain sensitive data)
- High-frequency loop iterations (log aggregates instead)
- Redundant position logs ("entering method", "exiting method")
- Info logs for every DB read in a hot path

---

## Request-scoped Logging

Prefer `logger.FromContext(ctx)` over `logger.L()` inside handlers and services so that request trace IDs and other context values are attached automatically.

```go
// In a handler or service method
log := logger.FromContext(ctx)
log.Info("account.test_started", zap.Int64("account_id", id))
```

---

## LegacyPrintf

`logger.LegacyPrintf(component, format, args...)` is only for code that hasn't been migrated to structured logging yet. New code must use `logger.L().With(...)` or `logger.FromContext(ctx)`.

```go
// Legacy — acceptable only in existing code that hasn't been migrated
logger.LegacyPrintf("handler.admin.ops_ws", "[OpsWS] upgrade failed: %v", err)
```
