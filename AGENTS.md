# AGENTS.md

Instructions for AI coding agents working on this repository.

## What this repo is

An [Agent Skill](https://agentskills.io) (`ts-agent-skills`) of standards for building JavaScript/TypeScript apps. The deliverable is Markdown, not code: there is no build, test suite or package to publish.

```
skills/ts-agent-skills/
├── SKILL.md          # core rules + table pointing to references
└── references/       # detailed guidance, loaded only when a task needs it
assets/               # banner.svg, icon.svg for the README
README.md             # user-facing docs and install instructions
```

## Editing rules

- **No library code snippets in the skill.** Rules, structure and naming only; agents verify library syntax in official docs. Folder trees are fine. Exception: dependency-free templates the project owns (e.g. `references/environment.md`) may include code; keep it type-checked under strict mode.
- **Version-agnostic.** Don't hard-code API names, config syntax or version numbers that change between releases; point to the official docs instead.
- **Keep `SKILL.md` lean** (well under 500 lines). Put detail in `references/` and add a row to the routing table in `SKILL.md` for any new reference file.
- **Frontmatter:** only `name`, `description` and `license`. `name` must match the folder name. Update `description` trigger words when adding a new framework or library.
- **One rule, one place.** Before adding a rule, search the skill for an existing one and edit it instead; don't let `SKILL.md` and `references/` contradict each other.
- **Opinionated defaults:** when adding or removing a preferred package, update the table in `SKILL.md`, the table in `README.md`, and the "Official docs" list.
- **Agent-neutral wording:** say "the agent", not a specific product, except where an instruction is genuinely product-specific.
- **Links:** use official docs or package READMEs, not blog posts. Check every new link resolves.

## Before finishing a change

1. Re-read the changed sections for contradictions with the rest of the skill.
2. Check the `SKILL.md` routing table matches the files in `references/`.
3. Keep `README.md` in sync (features list, defaults table, repo layout).
4. If `assets/banner.svg` changed, render it in a browser and check that the text and pills don't clip.

## Commits

Conventional Commits (`docs:`, `feat:` for new rules or references, `fix:` for corrections).

## Releasing

1. Bump `metadata.version` in each changed skill's `SKILL.md` (semver rules in `CHANGELOG.md`).
2. Add a `CHANGELOG.md` entry.
3. Validate locally: `pip install skills-ref`, then `agentskills validate skills/<name>`.
4. Commit, tag `vX.Y.Z` on `main`, push the tag, and create a GitHub Release from it.
