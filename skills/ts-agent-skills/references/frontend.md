# Frontend (React / Next.js)

## Components

- Function components with typed props interfaces; no `React.FC` needed.
- Keep components small; extract logic into custom hooks.
- Composition over prop drilling; context only for truly global, low-churn state.
- Stable, unique `key`s (never array index for dynamic lists).
- JSX attributes: an element with a single attribute stays on one line (`<div className="flex gap-2">`). An element with 2+ attributes puts each attribute on its own line, with the closing `>` or `/>` on its own line. Applies to every element, including self-closing ones.
  - Enforce via Prettier `singleAttributePerLine: true`. If the repo's Prettier config lacks it, suggest the developer add it rather than changing config unasked.

## Data fetching

Pick one server-state approach per app and don't mix them:

| App type | Use |
|---|---|
| Next.js app router | Server Components for reads, Server Actions for writes. Add TanStack Query only for client-heavy cases (polling, infinite lists, optimistic UI). |
| SPA using Redux Toolkit | RTK Query (see `redux.md`) |
| SPA without Redux | TanStack Query |
| Small, read-mostly app, or repo already on it | SWR |

- HTTP client: native `fetch` through one typed wrapper in `lib/http.ts` (base URL, auth header, timeout via `AbortSignal`, JSON parsing + schema validation, typed errors).
  - In Next.js, use `fetch` so the framework's caching and revalidation apply.
  - Use axios only when the repo already uses it or you need upload progress events.
- Define endpoints as one module per resource (`api/user.ts`) exporting typed functions; query hooks wrap those functions. Components never call `fetch` directly.
- Don't mirror server data into Redux slices or `useState`.

## Client state

- Local state first, then Zustand or Redux Toolkit for shared state (Redux: follow `redux.md`).
- Derive values during render; don't store derivable state. Avoid `useEffect` for data transforms or syncing state.
- Every effect with subscriptions, timers or fetches returns a cleanup.

## Next.js (app router)

- Server Components by default; `'use client'` only at interactive leaves.
- Fetch on the server; mark server-only modules with `import 'server-only'`. Only `NEXT_PUBLIC_*` env vars reach the browser.
- Validate Server Action inputs like any API endpoint; check auth inside each action.
- Use `loading.tsx`, `error.tsx`, `not-found.tsx`; set metadata per route.

## Forms

- React Hook Form + schema resolver (Zod). With MUI, use `@nish1896/rhf-mui-components` (see Opinionated defaults in SKILL.md).
- Show field-level errors; disable submit while pending; handle server errors.

## Performance

- Measure before memoizing. `useMemo` / `useCallback` / `memo` only for proven hot paths or referential-stability needs, or rely on React Compiler if enabled.
- Code-split routes and heavy widgets (`lazy`, `next/dynamic`); virtualize long lists; optimize images (`next/image`, explicit width/height, lazy loading).
- Watch bundle size; avoid importing whole libraries (`lodash` → per-function `lodash-es` or native).

## Accessibility & UX

- Semantic HTML first; every input has a label; buttons are `<button>`; keyboard and focus work; color contrast AA.
- Handle all states: loading, empty, error, success. No layout shift on load.

## Styling

- One styling system per project (Tailwind, MUI, CSS Modules).
- Use design tokens/theme from `src/theme`, not ad-hoc hex values (see `theme.md`).
- Tailwind: sort classes with `prettier-plugin-tailwindcss`. MUI: use `@nish1896/mui-components` or `@nish1896/rhf-mui-components` (see Opinionated defaults in SKILL.md).
