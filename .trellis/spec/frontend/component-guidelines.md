# Component Guidelines

> How components are built in this project.

---

## Overview

Vue3 Single File Components (SFCs) with `<script setup>` + TypeScript + Tailwind CSS.

---

## Component Structure

Standard order inside a `.vue` file:

```vue
<script setup lang="ts">
// 1. Imports
import { ref, computed } from 'vue'
import { useAppStore } from '@/stores/app'
import type { ApiKey } from '@/types'

// 2. Props / emits
interface Props {
  keyId: number
  readonly?: boolean
}
const props = defineProps<Props>()
const emit = defineEmits<{
  close: []
  saved: [key: ApiKey]
}>()

// 3. Store / composable usage
const appStore = useAppStore()

// 4. Reactive state
const loading = ref(false)

// 5. Computed
const isAdmin = computed(() => ...)

// 6. Methods / handlers
function handleSubmit() { ... }
</script>

<template>
  <!-- Single root element preferred -->
  <div class="...">
    ...
  </div>
</template>
```

---

## Props Conventions

- Always use `defineProps<Interface>()` with TypeScript generics — no runtime prop declarations
- Prop names: `camelCase` in script, `kebab-case` in template
- Boolean props: prefix with `is` or `has` (`isLoading`, `hasError`, `readonly`)
- Optional props use `?` in the interface definition

```ts
interface Props {
  accountId: number
  isLoading?: boolean
  onSuccess?: () => void
}
```

---

## Styling Patterns

- Tailwind CSS utility classes exclusively — no scoped `<style>` blocks unless unavoidable
- Dark mode: use `dark:` variants (e.g., `dark:from-dark-950`)
- Responsive: use `sm:`, `md:`, `lg:` prefixes

---

## Component Placement

- **Reusable, domain-agnostic** → `src/components/common/`
- **Feature-specific** → `src/components/{feature}/` (e.g., `components/keys/`, `components/account/`)
- **Page-level** → `src/views/`
- **Admin-only** → `src/components/admin/`

---

## Toast / User Feedback

Use `appStore.showSuccess()` / `appStore.showError()` for all user-facing notifications.
Format: "动作 + 结果" for success; "动作 + 结果, 原因" for failure.
Use the raw API `error.message` / `error.details` as the failure reason — do not rewrite it.

```ts
try {
  await api.keys.create(...)
  appStore.showSuccess('创建成功')
} catch (error: any) {
  const reason = error.response?.data?.message || error.message
  appStore.showError(`创建失败，${reason}`)
}
```

---

## Common Mistakes

- Using `<style scoped>` when Tailwind classes would suffice
- Calling API directly in template — always use composables or methods
- Putting business logic in `<template>` expressions — move to `computed` or methods
- Translating/rewriting backend error messages — always show the raw reason
