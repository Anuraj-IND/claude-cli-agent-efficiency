# Claude Code — Quick Reference Card

> ⚠️ READ-ONLY — Do NOT modify this file. This is a pre-built reference document.

> Key paths, commands, patterns, and configurations at a glance.

## Codebase Location

```
D:\cloude code\claude-code\
C:\cloude code\claude-code\
```

## Runtime & Build

```bash
bun run build          # Build CLI bundle
bun run build:web      # Build web frontend
bun run check          # Lint + typecheck
bun run test           # Run tests
bun run dev            # Development mode
```

- **Runtime**: Bun >=1.1.0 (NOT Node.js)
- **Language**: TypeScript strict, ESM with `.js` imports
- **Lint/Format**: Biome (tabs, single quotes, as-needed semicolons)

## Key Files

| File | Lines | Purpose |
|---|---|---|
| `src/entrypoints/cli.tsx` | — | CLI entry point |
| `src/main.tsx` | ~5,000 | Main CLI setup |
| `src/query.ts` | 1,245+ | AI query loop (async generator) |
| `src/QueryEngine.ts` | 1,297 | QueryEngine class wrapper |
| `src/Tool.ts` | 794 | Tool interface + buildTool factory |
| `src/tools.ts` | 396 | Tool registry |
| `src/commands.ts` | 758 | Command registry |
| `src/bootstrap/state.ts` | 1,577+ | Global state (DON'T ADD MORE) |
| `src/state/AppStateStore.ts` | 570 | Reactive state store |

## Key Directories

| Directory | Purpose |
|---|---|
| `src/tools/` | ~40 tool implementations |
| `src/commands/` | ~85 command implementations |
| `src/components/` | ~200 React/Ink components |
| `src/services/` | 18 service modules |
| `src/bridge/` | IDE bridge (25+ files) |
| `src/ink/` | Custom Ink framework |
| `web/` | Next.js web frontend |
| `mcp-server/` | MCP protocol server |
| `tests/` | Vitest test suite |
| `scripts/` | Build/dev/ops scripts |
| `docker/` | Docker configs |
| `helm/` | Kubernetes Helm chart |
| `grafana/` | Monitoring dashboards |

## Tool Categories

| Category | Key Tools |
|---|---|
| File System | Read, Write, Edit, Glob, Grep |
| Shell | Bash, PowerShell |
| Web | WebFetch, WebSearch |
| Agent | Agent, Task*, Team*, SendMessage |
| Planning | EnterPlanMode, ExitPlanMode, TodoWrite |
| MCP | ListMcpResources, ReadMcpResource, McpAuth |
| Special | REPL, Skill, Config, Sleep, LSP |

## Command Categories

| Category | Key Commands |
|---|---|
| Session | /clear, /compact, /resume, /session |
| Config | /config, /model, /theme, /permissions |
| Git | /diff, /branch, /commit, /review |
| MCP | /mcp |
| Auth | /login, /logout |
| Info | /help, /status, /stats, /cost |
| UI | /output-style, /keybindings, /vim |

## Feature Flags

```typescript
import { feature } from 'bun:bundle';

feature('PROACTIVE')        // Proactive suggestions
feature('COORDINATOR_MODE') // Multi-agent
feature('BRIDGE_MODE')      // IDE bridge
feature('VOICE_MODE')       // Voice
feature('HISTORY_SNIP')     // History snipping
feature('CONTEXT_COLLAPSE') // Context compression
```

## Permission Modes

| Mode | Description |
|---|---|
| `default` | Ask for destructive operations |
| `acceptEdits` | Auto-approve file edits |
| `dangerous` | Auto-approve except blocked |
| `fullAuto` | Auto-approve everything |

## AI Providers

| Provider | Env Vars |
|---|---|
| Anthropic | `ANTHROPIC_API_KEY` |
| AWS Bedrock | `ANTHROPIC_BEDROCK_ENABLED` |
| Google Vertex | `ANTHROPIC_VERTEX_ENABLED` |
| Azure Foundry | `ANTHROPIC_FOUNDRY_ENABLED` |

## State Layers

1. **Global** — `src/bootstrap/state.ts` (singleton, ~80 fields)
2. **AppState** — `src/state/AppStateStore.ts` (reactive, immutable updates)

## Database

- **ORM**: Drizzle
- **Local**: SQLite (`~/.claude/web/claude-code.db`)
- **Production**: PostgreSQL (via `DATABASE_URL`)

## MCP Server

- **8 Tools**: list_tools, list_commands, get_tool_source, get_command_source, read_source_file, search_source, list_directory, get_architecture
- **3 Resources**: architecture, tools, commands (+ source template)
- **5 Prompts**: explain_tool, explain_command, architecture_overview, how_does_it_work, compare_tools
- **Transports**: STDIO, HTTP/SSE, Vercel serverless

## Critical Rules

1. **DO NOT modify `src/`** — Read-only leaked source
2. **Keep changes small** — Single-purpose, reviewable
3. **Preserve patterns** — Match existing style
4. **Read before writing** — Always read target file first
5. **Validate** — Run `bun run check` after changes

## Reference Files

| File | When to Read |
|---|---|
| `CLAUDE-CODE-ARCHITECTURE.md` | Full architecture, data flow |
| `CLAUDE-CODE-TOOLS.md` | Tool system, schema, permissions |
| `CLAUDE-CODE-COMMANDS.md` | Command system, types, all commands |
| `CLAUDE-CODE-PATTERNS.md` | Coding conventions, patterns |
| `CLAUDE-CODE-NAVIGATION.md` | How to find things |
