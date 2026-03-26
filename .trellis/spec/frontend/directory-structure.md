# Directory Structure

> How frontend code is organized in this project.

---

## Overview

Vue3 + TypeScript + Vite + pnpm. Single-page app in `frontend/`.

---

## Directory Layout

```
frontend/
├── src/
│   ├── api/             # Axios API modules (one file per resource)
│   │   ├── client.ts    # Axios instance + interceptors (auth, token refresh)
│   │   ├── index.ts     # Re-exports
│   │   ├── keys.ts      # /keys endpoints
│   │   ├── auth.ts      # /auth endpoints
│   │   ├── groups.ts    # /groups endpoints
│   │   └── admin/       # Admin-only API modules
│   ├── components/      # Vue SFC components
│   │   ├── common/      # Shared reusable components
│   │   ├── admin/       # Admin UI components
│   │   ├── account/     # Account-related components
│   │   ├── keys/        # API key components
│   │   ├── charts/      # Chart/visualization components
│   │   └── icons/       # SVG icon components
│   ├── composables/     # Vue composables (useXxx pattern)
│   ├── stores/          # Pinia stores
│   │   ├── index.ts     # Store exports
│   │   ├── app.ts       # Global UI state (sidebar, toasts, loading)
│   │   ├── auth.ts      # Auth state (user, token)
│   │   └── *.ts         # Feature stores
│   ├── views/           # Page-level components (mapped to router)
│   ├── router/          # Vue Router configuration
│   ├── types/
│   │   ├── index.ts     # All shared TypeScript types/interfaces
│   │   └── global.d.ts  # Global ambient declarations
│   ├── i18n/            # Internationalization (vue-i18n)
│   ├── utils/           # Pure utility functions (no Vue deps)
│   ├── styles/          # Global CSS / Tailwind base styles
│   ├── main.ts          # App entry point
│   └── App.vue          # Root component
├── package.json
├── pnpm-lock.yaml       # Must be committed; use pnpm, never npm
└── vite.config.ts
```

---

## Module Organization

- API calls belong in `src/api/` — one module per resource group
- Business/UI logic shared across components belongs in `src/composables/`
- Global app state belongs in `src/stores/`
- All shared TypeScript types belong in `src/types/index.ts`
- Page components go in `src/views/`, sub-components in `src/components/{feature}/`

---

## Naming Conventions

- Component files: `PascalCase.vue` (e.g., `UseKeyModal.vue`)
- Composable files: `camelCase.ts` with `use` prefix (e.g., `useForm.ts`)
- Store files: `camelCase.ts` named by domain (e.g., `auth.ts`, `app.ts`)
- API modules: `camelCase.ts` named by resource (e.g., `keys.ts`, `groups.ts`)
- Type names: `PascalCase` interfaces and types
- Test files: placed in `__tests__/` directory alongside source

---

## Examples

- API module: `frontend/src/api/keys.ts`
- Composable: `frontend/src/composables/useForm.ts`
- Store: `frontend/src/stores/app.ts`
- Types: `frontend/src/types/index.ts`
- Axios client: `frontend/src/api/client.ts`
