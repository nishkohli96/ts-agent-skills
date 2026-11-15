# Changelog

All notable changes to the skills in this repo. Versions follow [Semantic Versioning](https://semver.org), adapted for skills:

- **Major:** a rule removed or reversed, changing the code agents produce (e.g. enums → `as const`).
- **Minor:** new rules, reference files or frameworks covered.
- **Patch:** wording, typos, link fixes.

Each release is a git tag (`vX.Y.Z`) on `main`, matching `metadata.version` in each changed skill's `SKILL.md`.

## [1.1.0](https://github.com/nishkohli96/ts-agent-skills/releases/tag/v1.1.0)

**Released - 16 November, 2026**

### ✨ Added

- Import rule: one import statement per module, using the folder barrel (`@/constants`, `@/lib`, …) instead of multiple deep imports from the same folder; files inside a folder import siblings by relative path.
- Barrel rule: statements in `index.ts` are ordered alphabetically by path, matching the folder's file order.

### 🔧 Changed

- Env validation uses a dependency-free template, `references/environment.md` (private typed string/number/enum readers that report all errors at once), instead of a schema library.

## [1.0.0](https://github.com/nishkohli96/ts-agent-skills/releases/tag/v1.0.0)

**Released - 11 November, 2026**

### ✨ Added

- `ts-agent-skills` skill: core rules in `SKILL.md` plus references for folder structure, Redux, theming, frontend, backend, and security & testing.
- Opinionated defaults: `@nish1896/eslint-flat-config`, `prettier-plugin-tailwindcss`, `@nish1896/rhf-mui-components`, `@nish1896/mui-components`.
- Install instructions for any Agent Skills–compatible agent via `npx skills`.
