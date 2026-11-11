# Redux layout (Redux Toolkit)

Use Redux Toolkit only: no hand-written action types, `switch` reducers or legacy `createStore`. Check the Redux Toolkit docs for current syntax.

```
src/store/
├── index.ts              # configureStore, RootState, AppDispatch, makeStore
├── hooks.ts              # typed useAppDispatch / useAppSelector
├── api/
│   ├── baseApi.ts        # createApi with fetchBaseQuery, tagTypes, auth headers
│   ├── userApi.ts        # baseApi.injectEndpoints per resource
│   └── orderApi.ts
└── slices/
    ├── auth/
    │   ├── slice.ts      # createSlice: state, reducers, extraReducers
    │   ├── selectors.ts  # plain + createSelector memoized selectors
    │   ├── thunks.ts     # createAsyncThunk (only for non-RTK Query async logic)
    │   └── types.ts      # state interface, payload types, enums
    └── ui/
        ├── slice.ts
        └── selectors.ts
```

## Rules

- `store/index.ts` exports a `makeStore` factory plus `AppStore`, `RootState` and `AppDispatch` types derived from it; `store/hooks.ts` exports pre-typed `useAppDispatch` / `useAppSelector`. Check the Redux Toolkit TypeScript docs for the current way to type these.
- Server data goes through RTK Query (`store/api`), not slices. Don't add TanStack Query or SWR alongside it. Slices hold client state only: auth session, UI, preferences, wizard drafts.
- Components use only `useAppDispatch` / `useAppSelector`, never the untyped hooks.
- Type every slice state with an interface and every action with `PayloadAction<T>`.
- Export the reducer and actions from `slice.ts`; components use selectors, never reach into `state.x.y` inline for shared logic.
- Use `createSelector` for derived data; `createEntityAdapter` for normalized collections.
- Invalidate caches with `providesTags` / `invalidatesTags`, not manual refetches.
- Keep state serializable: no class instances, Dates (store ISO strings), Maps or functions.
- Next.js app router: create the store per request with `makeStore()` inside a `'use client'` `StoreProvider` (`useRef`), never a module-level singleton. Import the provider directly, not via a barrel.
- A feature used in one place keeps state local or in context; add a slice only for state shared across distant components.
