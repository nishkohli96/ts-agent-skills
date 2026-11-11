# Theme layout

All design tokens and theme logic live in `src/theme/`. Tokens are importable anywhere (server or client); the provider is client-only, so keep them in separate files.

```
src/theme/
├── tokens.ts          # colors (light + dark), fonts, font sizes, spacing, radii, shadows, breakpoints, z-index
├── theme.ts           # builds the UI library theme from tokens (MUI theme, CSS variables, styled-components)
├── ThemeProvider.tsx  # 'use client'; context with mode + setMode, persistence, system sync
├── ThemeScript.tsx    # inline script that applies the theme before first paint (no flash)
├── useTheme.ts        # typed hook; throws outside the provider
├── types.ts           # ThemeMode enum, token types
├── constants.ts       # storage key, media query (plain module, safe for server imports)
└── index.ts           # server-safe exports only: tokens, types, constants, ThemeScript (import ThemeProvider and useTheme directly)
```

## Library support first

Use the UI library's built-in color-scheme support before writing a custom provider:
- MUI: built-in light/dark color schemes and CSS variables (see MUI theming docs for the installed version).
- Next.js without MUI: `next-themes`.
- Tailwind: tokens as CSS variables in the theme config, dark mode via a `data-theme` attribute variant (check the docs for the installed version's config format).

Write a custom provider only when none of these fits. Tokens and the folder layout apply either way.

## Custom provider requirements

- `types.ts`: `ThemeMode` string enum (Light, Dark, System), a `ResolvedMode` type (Light | Dark), and a `ColorTokens` interface.
- `tokens.ts`: color palettes typed as `Record<ResolvedMode, ColorTokens>`, plus typography, spacing, radii, shadows, breakpoints.
- `ThemeProvider.tsx` (`'use client'`): context exposing `mode`, `resolvedMode`, `setMode`. Validate the stored value with a type guard before using it. Listen to the OS color-scheme media query while in System mode and clean up the listener.
- `ThemeScript.tsx` (server-safe): a static inline script in `<head>` that sets `data-theme` and `color-scheme` on `<html>` before first paint; add `suppressHydrationWarning` on `<html>`.
- `constants.ts`: storage key and media query. Never import plain values from a `'use client'` file into server code; Next.js turns them into client references.
- `useTheme.ts`: typed hook that throws when used outside the provider.

## Rules

- Components never hardcode hex values, px spacing or font names; use tokens, theme values or CSS variables.
- Every color token has a light and a dark value; text/background pairs meet WCAG AA contrast (4.5:1 body, 3:1 large text).
- Default order: saved choice → OS preference → light. Offer Light / Dark / System; System follows OS changes live.
- Apply the theme before first paint (ThemeScript or the library's equivalent); reading storage only in `useEffect` causes a flash.
- Wrap `localStorage` access in try/catch.
- Wrap the app once at the root (`app/layout.tsx` or `main.tsx`); never nest ThemeProviders.
- Split `tokens.ts` into `colors.ts`, `typography.ts`, etc. once it grows past ~150 lines.
