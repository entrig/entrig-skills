# AGENTS.md

Guidance for AI coding agents working on this repository.

> `CLAUDE.md` is a symlink to this file.

## What this repo is

Distribution repo for Entrig agent skills. Skills follow the [Agent Skills Open Standard](https://agentskills.io/).

This repo ships **skills only**. The Entrig MCP server is configured separately by users — see the README for instructions. We do not bundle a plugin manifest.

## Repository structure

```
skills/
  {skill-name}/
    SKILL.md              ← required
    references/           ← progressive-disclosure files
      *.md
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

Each SDK skill is **self-contained**. It owns the full flow for its framework: pre-flight prerequisite check, SDK install, native setup, code wiring, and pointing the user at the Entrig MCP for notification creation. There is no shared "entrig" skill to defer to.

1. Create `skills/entrig-{framework}/` with `SKILL.md` + `references/`.
2. Read the SDK's `README.md` and any `bin/` setup scripts in the corresponding `entrig_packages/entrig-{framework}/` to ground the skill in real behavior.
3. Match the structure of `skills/entrig-flutter/`:
   - Pre-flight checklist (account, Supabase, FCM/APNs, API key)
   - Quick integration steps (install → native setup → init → register → listeners → verify)
   - "Creating notifications via the Entrig MCP" section with MCP-loaded vs MCP-not-loaded branches
   - Common mistakes table
   - "What this skill does NOT do" guardrails
4. Copy `references/dashboard-setup.md` and `references/mcp-setup.md` from `entrig-flutter/` — these are largely framework-agnostic. Adjust if needed.
5. Add framework-specific references for native setup, manual fallbacks, and common mistakes.
6. Keep the body under 200 lines.
7. Add a row to the top-level `README.md`.

If you find content getting duplicated across 3+ SDK skills with no per-framework variation, consider extracting it to a shared location at that point — but not before. Premature deduplication causes its own problems.

## What NOT to include in skill folders

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md

Skills contain only what an AI agent needs to do the job. Marketing and human onboarding live in the top-level `README.md`.

## Versioning

Bump `metadata.version` on every change to `SKILL.md` or any file in `references/`. Use semver.

## MCP server

Users configure the Entrig MCP server themselves (see top-level README). The skills should:

- Detect when MCP tools are not available and tell the user how to add the server (instead of improvising via REST API calls or SQL).
- Reference the MCP tools by name (`get_context`, `create_notification`, etc.) when they're available — see `skills/entrig/references/mcp-usage.md`.
