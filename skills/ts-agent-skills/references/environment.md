# Environment variables

Read `process.env` once at boot in `src/constants/environment.ts`. Use this file as the template so every project validates env the same way. It uses no libraries, so it doesn't depend on any package version.

## Rules

- This is the only file that reads `process.env`. Everything else imports `ENV_CONFIG` (through the `@/constants` barrel).
- The helpers stay private to this file: don't export them or move them to `utils`. Move them to a shared package only when several apps in a monorepo need them.
- Add one line to `ENV_CONFIG` per variable, using the reader that matches its type. Give a default only when a safe one exists. Never default secrets, connection strings, or values that switch behavior between environments (`NODE_ENV`): a missing value must fail validation, not silently fall back to development behavior.
- Values are read as strings (`readEnv`) or numbers (`readEnvNumber`); `NODE_ENV` is a plain string.
- Keep only the readers the project actually uses; delete unused ones, since `noUnusedLocals` and lint flag unused functions.
- Every new variable also goes in `.env.example`.
- Always pass the value as a literal property access (`process.env.API_URL`), never `process.env[name]`. Bundlers only inline env vars written literally, so a dynamic lookup is `undefined` in browser code.
- Next.js: only `NEXT_PUBLIC_*` values may be used in client code. Put them in a separate `constants/publicEnvironment.ts`, and keep server-only values in `constants/environment.ts`, imported from server code only.
- Vite: client values come from `import.meta.env.VITE_*` (also written literally), not `process.env`. Check the bundler's env docs for the installed version.

## Template

```ts
// src/constants/environment.ts
const errors: string[] = [];

// Each reader takes the variable's name (for error messages) and its value,
// passed as a literal `process.env.NAME` so bundlers can inline it.
function readEnv(name: string, value: string | undefined, fallback?: string): string {
  const raw = value?.trim();
  if (raw !== undefined && raw !== '') return raw;
  if (fallback !== undefined) return fallback;
  errors.push(`Missing required env var: ${name}`);
  return '';
}

function readEnvNumber(name: string, value: string | undefined, fallback?: number): number {
  const raw = readEnv(name, value, fallback?.toString());
  const parsed = Number(raw);
  if (raw !== '' && !Number.isFinite(parsed)) errors.push(`Env var ${name} must be a finite number`);
  return parsed;
}

export const ENV_CONFIG = Object.freeze({
  nodeEnv: readEnv('NODE_ENV', process.env.NODE_ENV),
  port: readEnvNumber('PORT', process.env.PORT, 3000),
  databaseUrl: readEnv('DATABASE_URL', process.env.DATABASE_URL),
});

if (errors.length > 0) {
  throw new Error(`Invalid environment:\n${errors.join('\n')}`);
}
```
