# Type Safety

> Type safety patterns in this project.

---

## Overview

TypeScript with strict mode. All shared types live in `src/types/index.ts`. No validation library (Zod/Yup) is used — types are used for compile-time safety only.

---

## Type Organization

All shared interfaces and types go in `src/types/index.ts`:

```ts
// src/types/index.ts

export interface User {
  id: number
  username: string
  email: string
  role: 'admin' | 'user'
  status: 'active' | 'disabled'
  // ...
}

export interface BasePaginationResponse<T> {
  items: T[]
  total: number
  page: number
  page_size: number
  pages: number
}

// API response wrapper (used by Axios interceptors)
export interface ApiResponse<T> {
  code: number
  message: string
  data: T
}
```

Locally-scoped types (used only within one component) can be defined inline in that file.

---

## API Type Conventions

- Field names in interfaces must match the backend JSON field names exactly — no renaming, no camelCase conversion
- Use `snake_case` for field names that match backend responses (`page_size`, `created_at`, etc.)
- Use `null` for nullable fields, `undefined` / `?` for optional fields

```ts
export interface ApiKey {
  id: number
  name: string
  group_id: number | null   // nullable
  expires_at?: string       // optional
  created_at: string
}
```

---

## Component Prop Types

Always use TypeScript generics for `defineProps` and `defineEmits`:

```ts
interface Props {
  userId: number
  isLoading?: boolean
}
const props = defineProps<Props>()

const emit = defineEmits<{
  close: []
  saved: [user: User]
}>()
```

---

## Common Patterns

```ts
// Paginated response
type PaginatedResponse<T> = BasePaginationResponse<T>

// Union types for status fields
type AccountStatus = 'active' | 'inactive' | 'error'

// Record for maps
type GroupRates = Record<number, number>

// Generic fetch with optional signal
interface FetchOptions {
  signal?: AbortSignal
}
```

---

## Forbidden Patterns

- `any` — use `unknown` + type narrowing, or define the proper type
- Non-null assertion `!` unless value is provably non-null in context
- `@ts-ignore` — fix the underlying type issue instead
- Renaming or camelCase-converting API response field names in types

---

## Type Imports

Use `import type` for type-only imports to avoid runtime overhead:

```ts
import type { User, ApiKey } from '@/types'
import type { AxiosResponse } from 'axios'
```
