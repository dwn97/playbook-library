---
name: react-patterns
description: Use when building or reviewing React components, including hooks, where to keep state, effects, lists, forms and fetching data.
license: MIT
compatibility: Client-side React 18 or 19 with function components and hooks.
metadata:
  status: "candidate"
  topic: "frontend"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "ecc"
  origin-url: "https://github.com/affaan-m/ECC"
  origin-commit: "b92f181b3b748d1abbc2afd94af6aa7d8c8fca29"
  origin-path: "skills/react-patterns/SKILL.md"
  approval: "none"
---

# React component patterns

Adapted from the upstream skill for client-side React. Server Components, server actions and framework routing are left out.

## Steps

1. **Check the setup first.** Read `package.json` for the React version and for a framework (Next.js, Remix), a router and a data library. This entry covers client-side React. Where a framework is present, its documentation decides data loading and routing.
2. **Compute during render what can be computed.** A value derived from props or state is a plain `const` in the component body. Do not keep it in a second `useState` that an effect keeps in step.
3. **Put state in the lowest place that works.**
   - One component uses it: `useState` in that component.
   - A parent and a few descendants: lift it to the nearest common ancestor and pass it down.
   - Distant branches, rarely changing (theme, signed-in user, locale): Context.
   - Shared and changing often: an external store, if the project has one.
   - Data from a server: the project's data library, if it has one.

   Most small apps need only the first two.
4. **Update state without changing it in place.** Make a new array or object every time: `[...items, x]`, `items.map(...)`, `{ ...user, name }`. When the next value depends on the previous one, use the updater form: `setCount(c => c + 1)`.
5. **Call hooks at the top level of the component only.** Never inside a condition, a loop or after an early return.
6. **Effects synchronise the component with something outside React:** a subscription, a timer, the DOM, a network request. They are not for derived data, and not for reacting to a user's action, which belongs in the event handler.
   - List every reactive value the effect reads in its dependency array.
   - Return a cleanup for every subscription, timer and listener.
   - When fetching in an effect without a data library, guard against a stale response: set an `ignore` flag in the cleanup (or use an `AbortController`), and keep loading and error state.
7. **Lists** need a stable `key` taken from the data, such as an id. The index is acceptable only for a list that never reorders, inserts or deletes.
8. **Forms.** Use controlled inputs (`value` plus `onChange`) when the value drives other UI or validation. Handle `onSubmit` and call `event.preventDefault()`. Every input has a label.
9. **Compose** with `children` and props. Extract a custom hook only when the same sequence of hooks appears in two components.
10. **Do not memoize by default.** `useMemo`, `useCallback` and `React.memo` go in when a measurement or a dependency chain calls for them.
11. Use semantic elements (`button`, `a`, `nav`, `main`) before a `div` with handlers.

## Verify

- The browser console shows no warnings while you use the app. Missing keys and dependency problems appear there. If the project has the `react-hooks` lint rules, run the linter.
- In development under `StrictMode`, React mounts each component a second time, so effects run twice. The UI must behave the same. A doubled request or a doubled listener means a missing cleanup.
- Change each piece of state through the UI and confirm every dependent part updates.
- Type quickly in any input that triggers fetching. The result shown must belong to the last input.
- Run the project's tests if it has them.

## Pitfalls

- Changing state in place (`items.push(x); setItems(items)`) does not re-render.
- An effect that sets a state value it also depends on loops forever.
- An object or function created during render and listed as a dependency is new on every render, so the effect runs every time.
- A timer or listener callback sees the state of the render that created it. Use the updater form or a ref to read the current value.
- Fetching in an effect with no guard shows whichever response arrives last, which is not always the latest request.
- Upstream also covers Server Components, React 19 form actions and `useOptimistic`. Read the source skill named in this entry's origin fields when a task needs them.
