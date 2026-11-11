# Folder structure

Default layout for a single JS/TS app. Follow the repo's existing structure if one exists. Group by layer at the top level; in larger apps, group by feature inside `routes` (backend) or `features` (frontend).

## Folders

- `public` - static assets served as-is (images, SVGs, fonts). Frontend apps, e.g. Next.js, Vite.
- `src` - all source code.
  - `components` - reusable UI components (frontend). `index.ts` re-exports server-safe components only; client components (`'use client'`) are imported by direct file path.
  - `constants` - app-wide constants and the validated env module (`environment.ts`, parsed at boot; see "Validate at every boundary" in SKILL.md).
  - `db` - database client, models/schemas and data-access layer (backend).
  - `lib` - wrappers around third-party services and SDKs (Redis, Stripe, analytics) and the typed HTTP client. Created once, imported everywhere.
  - `middleware` - reusable request middleware: auth, logging, error handling (backend).
  - `routes` - one folder per resource with its controller, service, validation schema and types (backend).
  - `store` - Redux Toolkit store, slices and RTK Query APIs (frontend; see `redux.md`).
  - `theme` - design tokens, theme setup and ThemeProvider (frontend; see `theme.md`).
  - `types` - shared TypeScript types and enums.
  - `utils` - small, pure, generic helpers with no business logic and no I/O.

## Generic layout

```
app-name/
├── public/
│   └── logo.svg
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Footer.tsx    # server component, re-exported from index.ts
│   │   │   └── NavMenu.tsx   # 'use client', imported directly, not via index.ts
│   │   ├── Button.tsx
│   │   └── index.ts          # server-safe components only
│   ├── constants/
│   │   ├── environment.ts
│   │   ├── preferences.ts
│   │   └── index.ts
│   ├── db/
│   │   ├── models/
│   │   │   ├── order.ts
│   │   │   ├── user.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── lib/
│   │   ├── http.ts           # typed fetch wrapper
│   │   ├── mixpanel.ts
│   │   ├── redis.ts
│   │   ├── stripe.ts
│   │   └── index.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── logger.ts
│   │   └── index.ts
│   ├── routes/
│   │   ├── user/
│   │   │   ├── controller.ts
│   │   │   ├── service.ts
│   │   │   ├── validation.ts
│   │   │   └── types.ts
│   │   └── index.ts
│   ├── store/                # see redux.md
│   ├── theme/                # see theme.md
│   ├── types/
│   │   ├── analytics.ts
│   │   ├── currency.ts
│   │   ├── filtering.ts
│   │   ├── user.ts
│   │   └── index.ts
│   └── utils/
│       ├── currency.ts
│       └── index.ts
├── .env                      # git-ignored, never committed
├── .env.example
├── .gitignore
├── .prettierignore
├── .prettierrc
├── eslint.config.mjs
├── package.json
└── tsconfig.json
```

## Next.js (app router) variant

Next.js owns routing through `src/app/`, so there is no `routes/` folder. Backend-only folders (`db`, server services) are used only from server code.

```
src/
├── app/
│   ├── (marketing)/          # route groups
│   ├── api/                  # route handlers
│   ├── layout.tsx
│   ├── page.tsx
│   ├── loading.tsx
│   ├── error.tsx
│   └── not-found.tsx
├── components/
├── features/                 # feature folders once a feature grows: orders/{api,components,hooks,types,utils}
├── hooks/                    # shared client hooks
├── lib/
├── store/
├── theme/
├── types/
└── utils/
```

Next's request-interception file (middleware/proxy) lives where the installed version's docs say; check before creating it.

## NestJS variant

Follow Nest's module layout instead of `routes/`:

```
src/
├── modules/
│   └── users/
│       ├── dto/
│       ├── entities/
│       ├── users.controller.ts
│       ├── users.module.ts
│       └── users.service.ts
├── common/                   # guards, filters, interceptors, pipes, decorators
├── config/                   # validated config module
├── app.module.ts
└── main.ts
```

## Rules

- Tests sit next to the file (`service.test.ts`) or in a top-level `tests/` folder; pick one per repo.
- Keep `index.ts` barrels small, at folder boundaries only. Deep barrels slow builds, break tree-shaking and cause circular imports.
- Never re-export client components (`'use client'` files) or client hooks from a shared barrel. Import them by direct path (`@/components/layout/NavMenu`). A mixed barrel pulls every re-exported module into whatever imports it: client bundles grow, and server-only code (DB, secrets, `server-only` imports) can leak into or break client builds.
- Never put `'use client'` at the top of an `index.ts`; it turns every re-export into a client component.
- If many client components need grouping, give them a separate barrel (e.g. `components/client/index.ts`) imported only by client code.
- Imports flow downward only; a lower layer never imports from a higher one:
  - Backend: `routes` / `middleware` → services → `db` / `lib` → `utils` / `types` / `constants`.
  - Frontend: `app` (pages) → `features` / `components` → `store` / `hooks` / `theme` → `lib` → `utils` / `types` / `constants`.
  - `utils`, `types` and `constants` import nothing from app layers.
