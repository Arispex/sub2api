# Hook Guidelines (Composables)

> How Vue composables are used in this project.

---

## Overview

Vue3 composables (equivalent to React hooks) live in `src/composables/`. They encapsulate reusable stateful logic with Vue's Composition API.

---

## Naming Conventions

- File name: `use{FeatureName}.ts` (e.g., `useForm.ts`, `useClipboard.ts`)
- Exported function: same name as file (e.g., `export function useForm(...)`)
- Always prefix with `use`

---

## Composable Structure

```ts
// src/composables/useClipboard.ts
import { ref } from 'vue'

export function useClipboard() {
  const copied = ref(false)

  async function copy(text: string): Promise<void> {
    await navigator.clipboard.writeText(text)
    copied.value = true
    setTimeout(() => { copied.value = false }, 2000)
  }

  return { copied, copy }
}
```

Return an object with named properties — not an array (unlike React's `useState`).

---

## useForm Pattern

The project has a shared `useForm` composable for standardized form submission:

```ts
// src/composables/useForm.ts
export function useForm<T>(options: {
  form: T
  submitFn: (data: T) => Promise<void>
  successMsg?: string
  errorMsg?: string
}) {
  const loading = ref(false)
  // manages loading, toast notifications, error propagation
  const submit = async () => { ... }
  return { loading, submit }
}
```

Use `useForm` for any form that calls an API — do not replicate the loading/error pattern manually.

---

## Data Fetching

There is no React Query or SWR equivalent. Data fetching is done:

1. In the component `onMounted` / `watch` directly using `async` functions and `ref` state
2. Or delegated to a Pinia store action for shared/cached data

```ts
// In component
const items = ref<ApiKey[]>([])
const loading = ref(false)

onMounted(async () => {
  loading.value = true
  try {
    items.value = await api.keys.list()
  } finally {
    loading.value = false
  }
})
```

For abort signal support, pass `{ signal }` from `AbortController` to API calls.

---

## Existing Composables

| Composable | Purpose |
|------------|---------|
| `useForm` | Form submission with loading, toast, error |
| `useClipboard` | Copy text to clipboard with feedback |
| `useKeyedDebouncedSearch` | Debounced search input |
| `useNavigationLoading` | Route change loading indicator |
| `useModelWhitelist` | Model whitelist management |
| `useAccountOAuth` | OAuth flow for account linking |
| `useOnboardingTour` | Onboarding tour state |

---

## Common Mistakes

- Writing the same loading/error pattern in multiple components instead of using `useForm`
- Naming composables without the `use` prefix — breaks Vue conventions
- Returning arrays instead of objects (makes destructuring fragile)
- Putting global side effects (API calls on import) inside composables — initialize lazily
