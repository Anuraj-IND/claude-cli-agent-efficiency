# Claude Code — Codebase Navigation Guide

> How to find anything in the Claude Code codebase quickly.

## Finding Things

### Find a Tool

```bash
# By name
rg "name: 'Bash'" src/tools/

# By directory
ls src/tools/BashTool/

# All tools
ls src/tools/
```

**Pattern**: Tools live in `src/tools/<ToolName>/` directories, each with an `index.ts` that exports the tool.

### Find a Command

```bash
# By name
rg "name: 'help'" src/commands/

# By directory
ls src/commands/help/

# All commands
ls src/commands/
```

**Pattern**: Commands live in `src/commands/<command-name>/` directories or are defined inline in `src/commands.ts`.

### Find a Component

```bash
# By name
rg "function ChatWindow" src/components/

# By file
ls src/components/chat/
```

**Pattern**: Components live in `src/components/<category>/<ComponentName>.tsx`.

### Find a Service

```bash
# All services
ls src/services/

# Specific service
ls src/services/mcp/
```

**Pattern**: Services live in `src/services/<service-name>/`.

### Find a Type

```bash
# By name
rg "interface AppState" src/

# In types directory
ls src/types/
```

**Pattern**: Core types in `src/types/`, component-specific types alongside their components.

---

## Key Search Patterns

### Find Tool Definitions

```bash
rg "buildTool\(" src/tools/
```

### Find Command Definitions

```bash
rg "type: 'local'" src/commands/
rg "type: 'local-jsx'" src/commands/
rg "type: 'prompt'" src/commands/
```

### Find Feature Flag Usage

```bash
rg "feature\('" src/
```

### Find API Calls

```bash
rg "api\." src/services/api/
rg "stream(" src/
```

### Find State Usage

```bash
rg "getState()" src/
rg "setState(" src/
rg "useAppState(" src/
```

### Find Permission Checks

```bash
rg "checkPermissions" src/
```

### Find React Components

```bash
rg "function [A-Z]" src/components/
```

### Find Ink Components

```bash
rg "from 'ink'" src/
```

---

## Study Paths

### Path 1: Understand the AI Loop

1. Read `src/entrypoints/cli.tsx` — Entry point
2. Read `src/main.tsx` — CLI setup (skim, focus on launchRepl)
3. Read `src/query.ts` — The core AI loop
4. Read `src/QueryEngine.ts` — Class wrapper
5. Read `src/Tool.ts` — Tool interface

### Path 2: Understand the Tool System

1. Read `src/Tool.ts` — Interface and buildTool
2. Read `src/tools.ts` — Registry
3. Read `src/tools/BashTool/index.ts` — Example tool
4. Read `src/tools/FileReadTool/index.ts` — Another example
5. Read `src/types/permissions.ts` — Permission types

### Path 3: Understand the UI

1. Read `src/screens/REPL.tsx` — Main screen
2. Read `src/ink/` — Ink framework
3. Read `src/components/design-system/` — Design system
4. Read `src/components/messages/` — Message components
5. Read `src/components/prompt-input/` — Input components

### Path 4: Understand Commands

1. Read `src/commands.ts` — Registry
2. Read `src/types/command.ts` — Command types
3. Read `src/commands/help/` — Example command
4. Read `src/commands/config/` — JSX command example
5. Read `src/commands/skills/` — Skill commands

### Path 5: Understand Services

1. Read `src/services/api/` — API client
2. Read `src/services/mcp/` — MCP client
3. Read `src/services/compact/` — Context compaction
4. Read `src/services/analytics/` — Analytics
5. Read `src/services/plugins/` — Plugin system

---

## File Size Reference

| File | Lines | Notes |
|---|---|---|
| `src/QueryEngine.ts` | ~46,000 | Largest file (includes bundled deps) |
| `src/Tool.ts` | ~29,000 | Tool system (includes bundled deps) |
| `src/commands.ts` | ~25,000 | Command system (includes bundled deps) |
| `src/main.tsx` | ~5,000 | Main CLI setup |
| `src/bootstrap/state.ts` | ~1,577 | Global state |
| `src/query.ts` | ~1,245 | AI loop |
| `src/QueryEngine.ts` | ~1,297 | QueryEngine class |
| `src/Tool.ts` | ~794 | Tool interface |
| `src/tools.ts` | ~396 | Tool registry |
| `src/commands.ts` | ~758 | Command registry |
| `src/types/message.ts` | ~415 | Message types |
| `src/types/permissions.ts` | ~442 | Permission types |
| `src/types/command.ts` | ~217 | Command types |

---

## MCP Server for Exploration

The MCP server at `mcp-server/` provides tools for exploring the codebase:

```
list_tools          — List all agent tools
list_commands       — List all slash commands
get_tool_source     — Read a tool's source file
get_command_source  — Read a command's source file
read_source_file    — Read any source file (with line range)
search_source       — Regex search across all files
list_directory      — List directory contents
get_architecture    — Get architecture overview
```

### Using MCP Tools

```
# List all tools
Call: list_tools

# Read a tool's source
Call: get_tool_source
Params: { toolName: "BashTool" }

# Search for a pattern
Call: search_source
Params: { pattern: "buildTool", maxResults: 20 }

# Read a specific file
Call: read_source_file
Params: { path: "tools/BashTool/index.ts", startLine: 1, endLine: 50 }
```

---

## Directory Tree (Top Level)

```
src/
├── entrypoints/       # CLI entry points
│   ├── cli.tsx        # Main entry
│   ├── init.ts        # Initialization
│   └── ...
├── main.tsx           # CLI setup
├── setup.ts           # Environment setup
├── query.ts           # AI loop
├── QueryEngine.ts     # QueryEngine class
├── Tool.ts            # Tool interface
├── tools.ts           # Tool registry
├── commands.ts        # Command registry
├── Task.ts            # Task types
├── bootstrap/         # Global state
│   └── state.ts
├── state/             # AppState store
│   └── AppStateStore.ts
├── types/             # TypeScript types
├── tools/             # Tool implementations (~40)
├── commands/          # Command implementations (~85)
├── components/        # React/Ink components (~200)
├── screens/           # Main screens (REPL, etc.)
├── ink/               # Ink framework
├── services/          # Service modules (18+)
├── bridge/            # IDE bridge (25+ files)
├── tasks/             # Task implementations
├── constants/         # Constants (18 files)
├── utils/             # Utility modules (100+)
├── hooks/             # React hooks
├── shims/             # Runtime shims
├── server/            # Web server
└── ...
```

---

## Common Tasks

### Add a New Tool

1. Create directory: `src/tools/MyTool/`
2. Create `index.ts` with `buildTool()` definition
3. Add to `src/tools.ts` in `getAllBaseTools()`
4. Run `bun run check`

### Add a New Command

1. Create directory: `src/commands/my-command/`
2. Create `index.ts` with command definition
3. Add to `src/commands.ts` in `COMMANDS()`
4. Run `bun run check`

### Add a New Component

1. Create file: `src/components/category/MyComponent.tsx`
2. Export from `src/components/category/index.ts`
3. Use in parent component
4. Run `bun run check`

### Modify Existing Tool

1. Find tool: `src/tools/<ToolName>/`
2. Read current implementation
3. Make changes
4. Run `bun run check`

### Modify Existing Command

1. Find command: `src/commands/<command-name>/` or `src/commands.ts`
2. Read current implementation
3. Make changes
4. Run `bun run check`

---

## Quick Grep Patterns

```bash
# Find where a function is defined
rg "function functionName" src/

# Find all imports of a module
rg "from './Module'" src/

# Find all uses of a type
rg ": TypeName" src/

# Find feature flag checks
rg "feature\('FLAG'\)" src/

# Find error handling
rg "catch.*error" src/

# Find async functions
rg "async function" src/

# Find React components
rg "function [A-Z]" src/components/

# Find tool calls
rg "\.call\(" src/

# Find API requests
rg "await api\." src/
```
