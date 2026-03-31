# Claude Code — AI Agent Reference

> **READ THIS FILE FIRST** — This is the entry point for any AI coding agent (Claude, GPT, Gemini, Cursor, opencode, etc.) working with the Claude Code codebase.

## ⚠️ READ-ONLY — DO NOT MODIFY

**All files in `D:\GLOBAL\` are READ-ONLY reference material.**
- Do NOT edit, rewrite, or modify any file in this directory
- Do NOT append content to these files
- Do NOT create new files in this directory
- These are pre-built reference documents — treat them as documentation, not workspace
- If you need to take notes, create them in your working project directory, NOT here

## What Is This

A centralized reference library for the **Claude Code** codebase — Anthropic's terminal-based AI coding assistant (~512K lines of TypeScript). All files here are designed to be read by AI agents from any CLI tool.

## File Index

All reference files are in this directory (`D:\GLOBAL\`):

| File | When to Read |
|---|---|
| `CLAUDE-CODE-ARCHITECTURE.md` | **Start here** — Full system architecture, data flow, entry points, subsystem map |
| `CLAUDE-CODE-QUICK-REF.md` | **Quick lookups** — Key paths, commands, patterns at a glance |
| `CLAUDE-CODE-TOOLS.md` | Tool system: schema, permissions, all 40+ tools with input/output schemas |
| `CLAUDE-CODE-COMMANDS.md` | Command system: types, all 85+ slash commands, definition patterns |
| `CLAUDE-CODE-PATTERNS.md` | Coding conventions, patterns, anti-patterns, best practices |
| `CLAUDE-CODE-NAVIGATION.md` | How to find anything in the codebase, study paths, grep patterns |

## Codebase Location

```
D:\cloude code\claude-code\
```

This directory contains the full Claude Code codebase (~512K lines of TypeScript).

## Critical Rules

1. **DO NOT modify `src/`** — The leaked source is preserved as-is. All changes go to tooling, docs, MCP server, web UI, scripts, and configs.
2. **Keep changes small** — Single-purpose, reviewable changes only.
3. **Preserve existing patterns** — Match existing code style, naming, and architecture.
4. **Read before writing** — Always read the relevant reference file AND the target file before making changes.
5. **Validate after changes** — Run `bun run check` (lint + typecheck) after any code modification.

## Runtime & Build

- **Runtime**: Bun (>=1.1.0), NOT Node.js
- **Language**: TypeScript strict mode, ESM with `.js` import extensions
- **Build**: `bun run build` (esbuild bundle), `bun run build:web` (Next.js)
- **Check**: `bun run check` (lint + typecheck)
- **Test**: `bun run test`

## Key Directories

```
src/            — Core application (512K lines, READ-ONLY)
web/            — Next.js web frontend
mcp-server/     — MCP protocol server for codebase exploration
tests/          — Vitest test suite
scripts/        — Build, dev, and operational scripts
docker/         — Docker Compose + Dockerfile configs
helm/           — Kubernetes Helm chart
grafana/        — Monitoring dashboards
docs/           — Detailed documentation
prompts/        — AI rebuild prompts
.github/        — CI/CD workflows
```

## Important Notes

- Source files use `.js` import extensions (ESM convention) but are `.ts`/`.tsx` files
- Feature flags use `feature('FLAG_NAME')` from `bun:bundle` virtual module
- The `bun:bundle` import is shimmed at runtime via `scripts/bun-plugin-shims.ts`
- Global state lives in `src/bootstrap/state.ts` — do not add more state there
- The main AI loop is in `src/query.ts` (async generator pattern)
- QueryEngine class wrapper is in `src/QueryEngine.ts`

## How to Use as an AI Agent

1. **Start here** — Read this file to understand the project structure
2. **Pick your reference** — Based on your task, read the appropriate `.md` file from this directory
3. **Navigate the codebase** — Use `CLAUDE-CODE-NAVIGATION.md` for find/search patterns
4. **Follow conventions** — Use `CLAUDE-CODE-PATTERNS.md` for coding style and patterns
5. **Validate** — Always run `bun run check` after making changes
