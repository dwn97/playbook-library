---
name: vue-patterns
description: Use when building or reviewing a Vue 3 app with the Composition API, including components, reactivity, Pinia state management and Vue Router.
license: MIT
compatibility: Vue 3 with script setup. Some steps need Vue 3.4 or 3.5, as marked. Pinia and Vue Router where the project uses them.
metadata:
  status: "candidate"
  topic: "frontend"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "ecc"
  origin-url: "https://github.com/affaan-m/ECC"
  origin-commit: "b92f181b3b748d1abbc2afd94af6aa7d8c8fca29"
  origin-path: "skills/vue-patterns/SKILL.md"
  approval: "none"
---

# Vue 3 app patterns

Adapted from the upstream skill for small and medium client-side apps. Nuxt, server rendering and performance directives are left out.

## Steps

1. **Check the versions first.** Read `package.json` for `vue`, `pinia`, `vue-router` and whether the project uses TypeScript. Some APIs below need Vue 3.4 or 3.5. In an existing codebase follow its style, Options API included, instead of converting it.
2. **Components use `<script setup>`,** in this order: imports, props and emits, composables, local state, computed values, functions, watchers, lifecycle hooks. Then `<template>`, then `<style scoped>`.
3. **Reactivity.**
   - `ref()` is the default. Use it for anything that may be replaced as a whole: primitives, arrays from an API, objects you reassign. Read and write through `.value` in script. Templates unwrap it.
   - `reactive()` only for an object you change in place and never reassign. Reassigning it or destructuring it disconnects the copy from the original.
   - A derived value is a `computed()`, never a watcher that writes another ref.
   - `watch(source, callback)` takes a ref or a getter. To watch a prop or a property, pass a getter: `watch(() => props.id, ...)`.
4. **Props go down, events go up.** Declare props with `defineProps` (types and defaults) and every event with `defineEmits`. Never change a prop. Two-way binding on a component is `defineModel()` from Vue 3.4, or `modelValue` plus `update:modelValue`.
   - In Vue 3.4 and below, a variable destructured from `defineProps()` is a constant that never updates. Use `props.x` or `toRefs(props)`.
   - From Vue 3.5 destructured props stay reactive. They still need a getter in `watch`: `watch(() => count, ...)`.
5. **Put state in the lowest place that works.**

   | who needs it | where it lives |
   |---|---|
   | one component | `ref` or `reactive` in that component |
   | a parent and its child | props and emits |
   | a subtree, rarely changing (theme, config) | `provide` and `inject` |
   | several pages or distant components | a Pinia store |

6. **Pinia stores** live in `stores/`, one per concern, in setup syntax: `defineStore("cart", () => { ... })`. Inside, a `ref` is state, a `computed` is a getter and a function is an action. Return every state property, or Pinia does not treat it as state.
   - In a component, take state and getters with `storeToRefs(store)`. Plain destructuring of state breaks reactivity.
   - Actions can be destructured straight from the store.
   - An async action tracks loading and error state as well as the result.
7. **Lists and conditions.** `v-for` needs a stable `:key` from the data, not the index. Do not put `v-if` and `v-for` on the same element; filter in a `computed`. Use `v-show` for something toggled often and `v-if` otherwise.
8. **Router.** Load route components lazily (`component: () => import("./pages/UserDetail.vue")`), navigate by route name, and guard with `router.beforeEach`, returning a route location to redirect. When only a param changes the component instance is reused, so watch `() => route.params.id` and fetch again.
9. **Forms.** `v-model` on inputs, `@submit.prevent` on the form, validation state as `computed`.
10. **Clean up** timers, listeners and subscriptions in `onUnmounted`, or in the watcher's cleanup callback.
11. Never use `v-html` with content a user can influence.

## Verify

- The app builds, and the browser console shows no Vue warnings while you use it. The warnings name most of the mistakes above.
- Change each piece of state through the UI and confirm every place that shows it updates. This is the test for lost reactivity.
- Move between two routes that differ only in a param and confirm the content changes.
- Reload the page on a deep route. It must still render.
- If the project has tests (usually Vitest with Vue Test Utils), run them. A store test starts from a fresh Pinia: `setActivePinia(createPinia())` before each test.

## Pitfalls

- A value that is right on first render and never updates is almost always lost reactivity: a destructured `reactive` object, a destructured store, or destructured props before Vue 3.5.
- A missing `.value` in script, or an added `.value` in a template.
- A store used at module scope, before the app has installed Pinia, throws. Call `useXStore()` inside `setup`, a function or a navigation guard.
- Mixing Options API and Composition API in one new component. Pick the one the codebase uses.
- Upstream also covers Nuxt, `v-memo`, `KeepAlive` and Vue 3.5 additions such as `useTemplateRef`, `onWatcherCleanup` and `useId`. Read the source skill named in this entry's origin fields when a task needs them.
