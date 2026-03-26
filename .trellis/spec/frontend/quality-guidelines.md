# Quality Guidelines

> Code quality standards for frontend development.

---

## Overview

Vue3 + TypeScript + pnpm. CI runs lint and type checks. Always use pnpm — never npm or yarn.

---

## Package Manager

Always use **pnpm**. Never use `npm install` or `yarn`.

```bash
cd frontend
pnpm install          # install dependencies
pnpm dev              # dev server
pnpm build            # production build
pnpm install --frozen-lockfile  # CI mode
```

If `pnpm-lock.yaml` is out of sync with `package.json`, run `pnpm install` and commit the updated lock file.

---

## PR Checklist

- [ ] `pnpm build` passes (no TypeScript errors)
- [ ] `pnpm-lock.yaml` committed if `package.json` changed
- [ ] No `any` types introduced
- [ ] User-facing messages follow "动作 + 结果" format; failure includes raw `error.message`
- [ ] API field names in types match backend JSON exactly (no renaming)
- [ ] New reusable logic extracted to a composable in `src/composables/`

---

## Forbidden Patterns

- `npm install` or `yarn` — always use pnpm
- `any` type — use proper TypeScript types
- Direct `console.log` left in production code
- Hardcoded strings that should come from i18n (if using i18n for that text)
- Custom toast/notification state in components — use `appStore.showSuccess/showError`
- Rewriting backend `error.message` in catch blocks — show the raw reason

---

## Required Patterns

- `<script setup lang="ts">` in all new components
- `import type` for type-only imports
- `useForm` composable for any form that calls an API
- `apiClient` from `src/api/client.ts` for all HTTP calls (handles auth token + interceptors)
- Types for all API request/response shapes defined in `src/types/index.ts`

---

## Testing

- Test files go in `__tests__/` directories alongside the code they test
- Test framework: Vitest (inferred from project structure)
- Tests are colocated by feature, not centralized

---

## Code Review Checklist

- API field names match backend exactly (no camelCase conversion of snake_case fields)
- Error messages shown to user use raw `error.response?.data?.message || error.message`
- No business logic in `<template>` — move to `computed` or methods
- New shared utilities added to appropriate directory (`composables/`, `utils/`, `stores/`)
- pnpm-lock.yaml updated if dependencies changed
