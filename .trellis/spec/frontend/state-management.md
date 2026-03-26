# State Management

> How state is managed in this project.

---

## Overview

State management uses **Pinia** with the Composition API style (`defineStore` + `setup` function). No Options API stores.

---

## Store Structure

```ts
// src/stores/app.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useAppStore = defineStore('app', () => {
  // State
  const loading = ref(false)
  const toasts = ref<Toast[]>([])

  // Computed
  const hasActiveToasts = computed(() => toasts.value.length > 0)

  // Actions
  function showSuccess(message: string): void { ... }
  function showError(message: string): void { ... }

  return { loading, toasts, hasActiveToasts, showSuccess, showError }
})
```

---

## State Categories

| Category | Where it lives | Example |
|----------|---------------|---------|
| Local component state | `ref()` / `reactive()` in component | form inputs, modal open state |
| Global UI state | `src/stores/app.ts` | sidebar, toasts, global loading |
| Auth state | `src/stores/auth.ts` | current user, token |
| Feature state | `src/stores/{feature}.ts` | cached lists, selected items |
| Server state | Component-local refs + store actions | fetched API data |

---

## When to Use Global State (Pinia)

Use a Pinia store when:
- Data is shared across multiple unrelated components
- Data needs to persist across route changes (e.g., user profile, public settings)
- Actions need to be triggered from anywhere (e.g., `showError`, `showSuccess`)

Use local `ref()` when:
- State is only used within one component or its children
- State resets when the component unmounts

---

## Existing Stores

| Store | ID | Purpose |
|-------|----|---------|
| `useAppStore` | `app` | Sidebar, toasts, loading indicator, public settings cache, version info |
| `useAuthStore` | `auth` | Current user, token, login/logout actions |
| `useAnnouncementsStore` | `announcements` | Announcements cache |
| `useSubscriptionsStore` | `subscriptions` | User subscriptions |
| `useAdminSettingsStore` | `adminSettings` | Admin config cache |
| `useOnboardingStore` | `onboarding` | Onboarding tour progress |

---

## Toast Notifications

Always use `appStore.showSuccess()` and `appStore.showError()` — never manage toast state locally.

```ts
const appStore = useAppStore()
appStore.showSuccess('保存成功')
appStore.showError(`保存失败，${error.message}`)
```

---

## Common Mistakes

- Using Options API store format (`state: () => ({...}), actions: {...}`) — use Composition API style
- Duplicating server-fetched data in multiple stores instead of fetching once and caching
- Calling `showError` with a rewritten message instead of the raw `error.message`
- Mutating store state directly from a component without going through an action
