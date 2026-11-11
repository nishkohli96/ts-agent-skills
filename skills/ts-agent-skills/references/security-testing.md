# Security and testing

## Security checklist

- Secrets only in env or a secret manager; `.env` git-ignored; ship `.env.example`.
- AuthN and authZ on every protected route, action and Server Action. Check ownership, not just login (prevent IDOR).
- Hash passwords with argon2 or bcrypt. Short-lived access tokens; `httpOnly`, `Secure`, `SameSite` cookies.
- Cookie-based auth: protect state-changing requests against CSRF (SameSite plus CSRF tokens or origin checks).
- `helmet`, a strict CORS allowlist, and rate limiting on auth and expensive endpoints.
- Escape output; never `dangerouslySetInnerHTML` with unsanitized input (use DOMPurify if unavoidable).
- Server fetching user-supplied URLs: guard against SSRF (allowlist hosts, block private/internal IP ranges, no redirects to internal addresses).
- Validate file uploads (type, size); store outside the web root or in object storage with signed URLs.
- No `eval`, `new Function`, or shell commands built from user input.
- `npm audit` / Dependabot; commit the lockfile; review new dependencies (maintenance, size, license).

## Testing

- Test pyramid: many unit tests (pure logic), fewer integration tests (API + real DB via Testcontainers or a test DB), few E2E tests (Playwright) for critical flows.
- Frontend: Testing Library; query by role/label; test behavior, not implementation. Mock the network with MSW.
- Backend: Jest or Vitest + Supertest; mock external services, not your own modules.
- Every bug fix gets a regression test. Cover edge cases: empty, null, boundaries, errors, concurrency.
- Tests are typed too; no `any` in mocks. Use the test runner's typed mock helpers or typed factories.
