# Environment variables

Read `process.env` once at boot in `src/constants/environment.ts`. Use this file as the template so every project validates env the same way. It uses no libraries, so it doesn't depend on any package version.

## Rules

- This is the only file that reads `process.env`. Everything else imports `ENV_CONFIG` (through the `@/constants` barrel).
- The helpers stay private to this file: don't export them or move them to `utils`. Move them to a shared package only when several apps in a monorepo need them.
- Add one line to `ENV_CONFIG` per variable, using the reader that matches its type. Give a default only when a safe one exists; secrets and connection strings never get one.
- Fixed sets of values (environments, log levels) use a string enum and `readEnvEnum`.
- Every new variable also goes in `.env.example`.
- Next.js: only `NEXT_PUBLIC_*` values may be used in client code. Keep server-only values in a module imported from server code only.

## Template

```ts
// src/constants/environment.ts
const errors: string[] = [];

function readEnv(name: string, fallback?: string): string {
  const raw = process.env[name]?.trim();
  if (raw !== undefined && raw !== '') return raw;
  if (fallback !== undefined) return fallback;
  errors.push(`Missing required env var: ${name}`);
  return '';
}

function readEnvNumber(name: string, fallback?: number): number {
  const raw = readEnv(name, fallback?.toString());
  const value = Number(raw);
  if (raw !== '' && Number.isNaN(value)) errors.push(`Env var ${name} must be a number`);
  return value;
}

function readEnvEnum<T extends string>(name: string, allowed: readonly T[], fallback?: T): T {
  const raw = readEnv(name, fallback);
  const match = allowed.find((option) => option === raw);
  if (match === undefined) {
    errors.push(`Env var ${name} must be one of: ${allowed.join(', ')}`);
    return allowed[0] as T; // placeholder only: the throw below stops the app first
  }
  return match;
}

export enum NodeEnv {
  Development = 'development',
  Test = 'test',
  Production = 'production',
}

export const ENV_CONFIG = Object.freeze({
  nodeEnv: readEnvEnum('NODE_ENV', Object.values(NodeEnv), NodeEnv.Development),
  port: readEnvNumber('PORT', 3000),
  databaseUrl: readEnv('DATABASE_URL'),
});

if (errors.length > 0) {
  throw new Error(`Invalid environment:\n${errors.join('\n')}`);
}
```
