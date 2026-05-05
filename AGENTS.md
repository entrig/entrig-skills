# AGENTS.md

Guidance for AI coding agents working on this repository.

> `CLAUDE.md` is a symlink to this file.

## What this repo is

Distribution repo for Entrig agent skills + bundled MCP server. Skills follow the [Agent Skills Open Standard](https://agentskills.io/).

## Repository structure

```
skills/
  {skill-name}/
    SKILL.md              ← required
    references/           ← progressive-disclosure files
      *.md
.mcp.json                 ← bundles entrig MCP — installed alongside skills
.claude-plugin/           ← Claude Code plugin manifest
.cursor-plugin/           ← Cursor plugin manifest
```

## Writing a SKILL.md

Required YAML frontmatter:

```yaml
---
name: skill-name           # must match directory name; lowercase, hyphens only
description: >             # 1-1024 chars. Triggers + what it does. CRITICAL.
  What it does, AND when to use it. List concrete triggers (frameworks,
  product names, user phrases) so the agent knows when to activate.
metadata:
  author: entrig
  version: "0.1.0"         # bump on every edit to SKILL.md or references/
---
```

**Description is the trigger.** Only the description is loaded for skill matching — the body loads only after the agent decides to activate. Stuff every relevant trigger keyword into the description.

**Body should be lean** — under 500 lines, ideally under 5k tokens. The body is a router: quick start + a "What do you need?" table linking to references. Move detailed content to `references/`.

**Imperative voice.** "Add the dependency" not "You should add the dependency."

**Don't write things the agent already knows** — no `npm install` examples, no full `main.dart` skeletons, no explanations of standard concepts.

## Writing references

References are loaded on demand. Use them for:

- Detailed setup steps for one platform
- Manual-fallback procedures when a CLI fails
- Troubleshooting tables
- Common-mistakes tables

Keep references narrow and focused. One file per concern.

## Adding a new SDK skill

1. Create `skills/entrig-{framework}/` with `SKILL.md` + `references/`.
2. Read the SDK's `README.md` and any `bin/` setup scripts in the corresponding `entrig_packages/entrig-{framework}/` to ground the skill in real behavior.
3. Match the structure of `skills/entrig-flutter/` — quick-start section, integration steps, common-mistakes table, references for platform-specific gotchas.
4. Keep it under 200 lines in the body.
5. Add an entry to `README.md` and to `skills/entrig/SKILL.md` (the router).

## What NOT to include in skill folders

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md

Skills contain only what an AI agent needs to do the job. Marketing and human onboarding live in the top-level `README.md`.

## Versioning

Bump `metadata.version` on every change to `SKILL.md` or any file in `references/`. Use semver.

## MCP server

The `.mcp.json` at the repo root configures the Entrig MCP server. When installed via `npx skills add`, this is auto-wired. Skills should reference the MCP server by name (`entrig`) when they want the agent to call MCP tools — see `skills/entrig/references/mcp-usage.md`.
