# Claude Code Architecture Reference

> ⚠️ READ-ONLY — Do NOT modify this file. This is a pre-built reference document.

> Full system architecture, data flow, entry points, and subsystem map for the Claude Code codebase.

## System Overview

Claude Code is a terminal-based AI coding assistant built with Bun + TypeScript + React/Ink. It runs an async AI loop that processes user input, calls the Anthropic API, executes tool calls, and renders output in the terminal.

**Scale**: ~512,000 lines of TypeScript across ~1,900 files.

**Codebase path**: `D:\cloude code\claude-code\` (also `C:\cloude code\claude-code\`)

---

## Pipeline Model

```
User Input
    │
    ▼
┌─────────────┐
│ CLI Parser  │  ← commands.ts (slash commands like /help, /config)
└──────┬──────┘
       │
       ▼
┌──────────────┐
│ Query Engine │  ← query.ts (async generator, the core AI loop)
│              │
│ 1. Skill     │
│    prefetch  │
│ 2. Compact   │
│ 3. API call  │
│ 4. Tool exec │
│ 5. Recovery  │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│ LLM API      │────▶│ Tool System  │
│ (streaming)  │     │ (~40 tools)  │
└──────┬───────┘     └──────┬───────┘
       │                    │
       ▼                    ▼
┌──────────────┐     ┌──────────────┐
│ Terminal UI  │     │ Task System  │
│ (React/Ink)  │     │ (6 types)    │
└──────────────┘     └──────────────┘
```

---

## Entry Points

### CLI Entry: `src/entrypoints/cli.tsx`
Fast-path dispatcher that checks argv before loading heavy modules. Routes to:
- `--version`, `--dump-system-prompt`
- `claude ssh`, `claude remote-control`, `claude daemon`
- `claude ps/logs/attach/kill`
- Template jobs, environment-runner, self-hosted-runner, tmux worktree
- Falls through to `main.tsx` for the full interactive CLI

### Main Entry: `src/main.tsx` (~5,000 lines)
Massive Commander.js CLI setup. Key responsibilities:
- Defines all CLI flags and subcommands
- Startup profiling with checkpoints
- MDM prefetch, keychain prefetch, early input capture
- Runs database migrations (version 11)
- Initializes GrowthBook feature flags
- Launches REPL via `launchRepl()` or headless mode via `QueryEngine`
- Handles `--remote`, `--worktree`, `--bare`, `--print` modes
- Feature-flagged paths: `COORDINATOR_MODE`, `KAIROS`, `BRIDGE_MODE`, `VOICE_MODE`

### Initialization: `src/entrypoints/init.ts`
Memoized `init()` function:
- Loads configs, safe env vars
- Sets up graceful shutdown
- 1P event logging, OAuth, JetBrains detection
- mTLS, proxy agents, API preconnect
- Upstream proxy (CCR), scratchpad

### Setup: `src/setup.ts`
- Node version check, UDS messaging
- Terminal backup restoration, worktree creation
- Hooks config snapshot, file-changed watcher
- LSP manager, attribution hooks
- Session file access hooks, team memory watcher
- Release notes, permission safety checks
- Previous session cost logging

---

## State Management

### Layer 1: Global Bootstrap State
**File**: `src/bootstrap/state.ts` (1,577+ lines)

Single mutable `STATE` object with ~80+ fields:
- Session ID, cost, duration, model usage
- Telemetry, hooks, channels
- Scroll drain suspension, interaction time batching
- Token budget tracking, session switching

**WARNING**: Contains comments "DO NOT ADD MORE STATE HERE" and "ALSO HERE - THINK THRICE BEFORE MODIFYING".

### Layer 2: AppState Store
**File**: `src/state/AppStateStore.ts` (570 lines)

- `AppState` type: massive discriminated union with `DeepImmutable<>` wrapper
- Contains: settings, model, tasks, MCP, plugins, todos, notifications, speculation, bridge state, team context, tungsten (tmux), bagel (web browser), computer use, inbox, worker permissions, ultraplan, fast mode, advisor model
- `getDefaultAppState()` — factory for initial state
- Store pattern: minimal pub/sub with `getState`, `setState` (updater function), `subscribe`

---

## The AI Query Loop

### Core: `src/query.ts` (1,245+ lines)
Async generator — the heart of the AI interaction loop.

```typescript
query(): AsyncGenerator<StreamEvent | Message | TombstoneMessage | ToolUseSummaryMessage>
```

**Per-iteration pipeline**:
1. **Skill discovery prefetch** — finds relevant skills for the current context
2. **Tool result budget enforcement** — limits tool output size
3. **Snip compact** — removes old history to stay within context limits
4. **Microcompact** — compresses recent messages
5. **Context collapse projection** — estimates if context will overflow
6. **Auto-compact** — proactively compacts when needed
7. **API streaming** — calls the model and streams response
8. **Tool execution** — runs tool calls via `StreamingToolExecutor`
9. **Recovery paths**:
   - prompt-too-long → collapse drain → reactive compact
   - max_output_tokens → escalation

**Feature flags**: `HISTORY_SNIP`, `CONTEXT_COLLAPSE`, `REACTIVE_COMPACT`, `CACHED_MICROCOMPACT`, `TOKEN_BUDGET`

### QueryEngine: `src/QueryEngine.ts` (1,297 lines)
Class wrapper around `query()` for headless/SDK mode:
- One `QueryEngine` per conversation
- `submitMessage()` — processes input, runs query loop, maps to SDK messages
- Handles: transcript recording, snip replay, usage accumulation, structured output, budget checks, permission denial tracking
- `ask()` — convenience wrapper for one-shot usage

---

## Tool System

### Tool Interface: `src/Tool.ts` (794 lines)
Generic interface `Tool<Input, Output, Progress>` with:

**Required methods**:
- `call()` — execute the tool
- `description()` — tool description for the model
- `inputSchema` — Zod schema for input validation
- `prompt()` — prompt text for the model
- `userFacingName()` — display name
- `renderToolUseMessage()` — Ink component for tool use display
- `checkPermissions()` — permission check
- `mapToolResultToToolResultBlockParam()` — result formatting

**Optional methods** (30+):
`isEnabled`, `isConcurrencySafe`, `isReadOnly`, `isDestructive`, `interruptBehavior`, `isSearchOrReadCommand`, `isOpenWorld`, `requiresUserInteraction`, `shouldDefer`, `alwaysLoad`, `backfillObservableInput`, `validateInput`, `preparePermissionMatcher`, `getToolUseSummary`, `getActivityDescription`, `toAutoClassifierInput`, `renderToolResultMessage`, `extractSearchText`, `isResultTruncated`, `renderToolUseTag`, `renderToolUseProgressMessage`, `renderToolUseQueuedMessage`, `renderToolUseRejectedMessage`, `renderToolUseErrorMessage`, `renderGroupedToolUse`

### Tool Registry: `src/tools.ts` (396 lines)
- `getAllBaseTools()` — exhaustive list of all built-in tools
- `getTools()` — filters by mode (simple mode = Bash/Read/Edit only)
- `assembleToolPool()` — combines built-in + MCP tools with dedup

### Tool Categories (~40 tools)

| Category | Tools |
|---|---|
| **File System** | Read, Write, Edit, Glob, Grep, FileEdit, FileWrite, FileRead |
| **Shell** | Bash, PowerShell, TerminalCapture |
| **Web** | WebFetch, WebSearch |
| **Agent/Orchestration** | Agent, TaskCreate, TaskGet, TaskUpdate, TaskList, TaskOutput, TaskStop, TeamCreate, TeamDelete, SendMessage |
| **Planning** | EnterPlanMode, ExitPlanMode, TodoWrite, Brief, VerifyPlan |
| **MCP** | ListMcpResources, ReadMcpResource, McpAuth |
| **Special** | REPL, Skill, Config, Sleep, ScheduleCron, LSP, NotebookEdit |
| **Advanced** | ToolSearch, SyntheticOutput, Workflow |

### Tool Definition Pattern

```typescript
const myTool = buildTool({
  name: 'MyTool',
  description: 'Does something useful',
  inputSchema: z.object({
    param1: z.string().describe('Description for the model'),
    param2: z.number().optional(),
  }),
  call: async ({ param1, param2 }, context) => {
    // Implementation
    return { content: [{ type: 'text', text: 'result' }] };
  },
  // Optional overrides
  isReadOnly: true,
  checkPermissions: (input, context) => {
    return { type: 'autoApprove' };
  },
});
```

---

## Command System

### Command Registry: `src/commands.ts` (758 lines)
- `COMMANDS()` — memoized array of ~60+ built-in commands
- `INTERNAL_ONLY_COMMANDS` — ant-only commands
- Feature-flagged commands: proactive, brief, assistant, bridge, remoteControlServer, voice, force-snip, workflows, ultraplan, torch, peers, fork, buddy, subscribe-pr
- `getCommands()` — loads all sources, filters by availability and enabled state
- `REMOTE_SAFE_COMMANDS` / `BRIDGE_SAFE_COMMANDS` — allowlists for remote mode

### Command Types: `src/types/command.ts` (217 lines)

| Type | Description | Example |
|---|---|---|
| `prompt` | Expands to text sent to the model | `/skill`, custom skills |
| `local` | Executes locally, returns text | `/help`, `/status` |
| `local-jsx` | Renders Ink UI components | `/config`, `/model`, `/theme` |

### Command Categories (~85 commands)

| Category | Commands |
|---|---|
| **Session** | /clear, /compact, /resume, /session, /rename, /rewind, /export, /copy |
| **Config** | /config, /model, /theme, /permissions, /hooks, /memory, /skills, /plugins |
| **Git/PR** | /diff, /branch, /commit, /commit-push-pr, /review, /pr_comments |
| **MCP** | /mcp (add, list, manage servers) |
| **Auth** | /login, /logout, /install-github-app, /install-slack-app |
| **Info** | /help, /status, /stats, /cost, /usage, /doctor, /context |
| **UI** | /output-style, /keybindings, /vim, /statusline, /stickers, /color |
| **Feature** | /desktop, /mobile, /chrome, /ide, /teleport, /bridge, /btw, /plan, /fast, /passes, /effort, /feedback |
| **Internal** | /ant-trace, /perf-issue, /mock-limits, /reset-limits, /good-claude, /issue, /bughunter, /ctx_viz, /debug-tool-call, /summary, /share |

---

## UI Layer (React/Ink)

### Ink Framework: `src/ink/`
Custom Ink fork with:
- Focus management, terminal viewport, tab status
- Terminal focus, synchronized output
- DEC control sequences, frame timing, flicker detection
- Components: Box, Text, Button, Link, Spacer, Newline, NoSelect, Ansi, RawAnsi
- Hooks: useInput, useApp, useStdin, useSelection, useAnimationFrame, useInterval, useTerminalViewport, useTerminalFocus, useTerminalTitle

### Design System: `src/components/design-system/`
ThemeProvider, ThemedBox, ThemedText, Dialog, Pane, Tabs, ListItem, FuzzyPicker, ProgressBar, StatusIcon, LoadingState, Divider, Ratchet, KeyboardShortcutHint, Byline, color utilities

### Main Screens: `src/screens/`
- **REPL.tsx** — main interactive screen (the primary UI)
- **ResumeConversation.tsx** — session picker
- **Doctor.tsx** — diagnostics screen

### Component Organization: `src/components/` (~200+ components)

| Category | Count | Examples |
|---|---|---|
| **Messages** | 25+ | AssistantText, AssistantThinking, UserPrompt, UserCommand, UserToolResult |
| **Permissions** | 20+ | Bash, FileEdit, FileWrite, WebFetch, ComputerUse dialogs |
| **PromptInput** | 15+ | History search, suggestions, footer, mode indicator |
| **Agents** | 10+ | Agent list, editor, detail, new-agent wizard |
| **MCP** | 8+ | Server list, tool detail, capabilities, reconnect dialogs |
| **Tasks** | 5+ | Background tasks, shell detail, remote session progress |
| **Spinner** | 6+ | Glimmer, Shimmer, Teammate tree animations |
| **Settings** | 5+ | Config, Status, Usage panels |
| **Feedback** | 4+ | Survey, transcript share prompts |
| **LogoV2** | 4+ | Animated logo, feed, channels notice, upsells |

---

## Services Layer

| Service | Path | Purpose |
|---|---|---|
| **API** | `src/services/api/` | Claude API client, error handling, retry, streaming, bootstrap data, files API |
| **Analytics** | `src/services/analytics/` | GrowthBook feature flags, 1P event logging, analytics config, sink management |
| **MCP** | `src/services/mcp/` | Client management, server config, official registry, channel allowlist, elicitation handler |
| **Compact** | `src/services/compact/` | Auto-compact, reactive compact, microcompact, snip compact, snip projection |
| **Plugins** | `src/services/plugins/` | Plugin loading, marketplace, versioned caches, hot reload |
| **SessionMemory** | `src/services/sessionMemory/` | Per-session memory files |
| **PromptSuggestion** | `src/services/promptSuggestion/` | AI-generated follow-up suggestions |
| **LSP** | `src/services/lsp/` | Language server protocol manager |
| **x402** | `src/services/x402/` | Payment protocol |
| **OAuth** | `src/services/oauth/` | OAuth client flow |
| **Tips** | `src/services/tips/` | Contextual tip registry |
| **AutoDream** | `src/services/autoDream/` | Background task automation |
| **ToolUseSummary** | `src/services/toolUseSummary/` | Tool activity summaries |
| **RemoteManagedSettings** | `src/services/remoteManagedSettings/` | Enterprise settings |
| **PolicyLimits** | `src/services/policyLimits/` | Org policy enforcement |
| **SettingsSync** | `src/services/settingsSync/` | Cross-device settings sync |
| **TeamMemorySync** | `src/services/teamMemorySync/` | Team memory synchronization |

---

## Bridge System (Remote Control)

**Directory**: `src/bridge/` (25+ files)

Enables remote IDE integration (VS Code, JetBrains):

| File | Purpose |
|---|---|
| `bridgeMain.ts` | Main entry point |
| `bridgeApi.ts` | API abstraction |
| `bridgeMessaging.ts` | Message protocol |
| `replBridge.ts` | REPL bridge |
| `remoteBridgeCore.ts` | Core bridge logic |
| `sessionRunner.ts` | Session management |
| `codeSessionApi.ts` | Code session API |
| `jwtUtils.ts` | JWT auth |
| `trustedDevice.ts` | Trusted device auth |
| `workSecret.ts` | Work secret management |
| `capacityWake.ts` | Wake-on-capacity |
| `inboundMessages.ts` | Inbound message handling |
| `bridgeUI.ts` | UI integration |
| `bridgePermissionCallbacks.ts` | Permission handling over bridge |

---

## Task System

### Task Types: `src/Task.ts` (127 lines)

```typescript
type TaskType =
  | 'local_bash'
  | 'local_agent'
  | 'remote_agent'
  | 'in_process_teammate'
  | 'local_workflow'
  | 'monitor_mcp'
  | 'dream';

type TaskStatus = 'pending' | 'running' | 'completed' | 'failed' | 'killed';

interface Task {
  name: string;
  type: TaskType;
  kill(): void;
}
```

Task ID generation with type prefixes: `b` (bash), `a` (agent), `r` (remote), `t` (teammate), `w` (workflow), `m` (monitor), `d` (dream).

---

## Permission System

### Permission Modes
| Mode | Description |
|---|---|
| `default` | Ask for permission on destructive operations |
| `acceptEdits` | Auto-approve file edits, ask for everything else |
| `dangerous` | Auto-approve everything except explicitly blocked |
| `fullAuto` | Auto-approve everything |

### Permission Rules
- Wildcard support for file paths and commands
- Per-tool permission checks via `checkPermissions()`
- Permission denial tracking and reporting

---

## Feature Flags

### Build-time Flags (`bun:bundle`)
```typescript
import { feature } from 'bun:bundle';

if (feature('PROACTIVE')) {
  // Proactive suggestions
}
```

### Known Flags
| Flag | Purpose |
|---|---|
| `PROACTIVE` | Proactive code suggestions |
| `COORDINATOR_MODE` | Multi-agent coordination |
| `BRIDGE_MODE` | IDE bridge integration |
| `VOICE_MODE` | Voice interaction |
| `KAIROS` | Internal feature |
| `HISTORY_SNIP` | History snipping in context |
| `CONTEXT_COLLAPSE` | Context compression |
| `REACTIVE_COMPACT` | Reactive message compaction |
| `CACHED_MICROCOMPACT` | Cached micro-compact results |
| `TOKEN_BUDGET` | Token budget tracking |

### Runtime Gates
- GrowthBook feature flags (remote-controlled)
- Environment variable checks (`USER_TYPE === 'ant'`, `NODE_ENV === 'test'`)
- Conditional imports via `require()` for dead code elimination

---

## Key Design Patterns

### 1. Feature Flagging
- `feature('FLAG_NAME')` from `bun:bundle` — compile-time dead code elimination
- Runtime gates via GrowthBook
- Env var checks
- Conditional imports via `require()`

### 2. Lazy Loading
- Dynamic `import()` everywhere for heavy components
- `require()` for feature-gated modules (avoids static analysis)
- Memoized lazy factories

### 3. Performance Optimizations
- Startup profiler with checkpoints throughout
- MDM/keychain prefetch at module load time
- Early input capture before module loading
- Deferred prefetches after first render
- Scroll drain suspension
- Interaction time batching (deferred `Date.now()`)
- Content-hash-based temp file paths (prompt cache stability)
- Reservoir sampling for stats histograms
- Fire-and-forget for non-critical async work

### 4. Error Handling
- Fail-open philosophy throughout
- Graceful degradation on plugin/skill failures
- Recovery loops in query (prompt-too-long, max-output-tokens)
- Circuit breakers for auto-compact failures
- Ring buffer for in-memory errors (100 max)

### 5. State Architecture
- Two-layer state: global singleton (bootstrap) + reactive store (AppState)
- Immutable state updates via updater functions
- `DeepImmutable` type wrapper
- `onChangeAppState` callback for side effects

---

## Build System

### Runtime
- **Bun** (>=1.1.0), NOT Node.js
- ESM with `.js` import extensions in `.ts`/`.tsx` files
- `bun:bundle` virtual module for feature flags

### Build Commands
```bash
bun run build          # esbuild bundle
bun run build:watch    # Watch mode
bun run build:prod     # Minified production bundle
bun run build:web      # Next.js web frontend
bun run check          # Lint + typecheck
bun run test           # Vitest
```

### Build Script: `scripts/build-bundle.ts`
- esbuild-based bundler
- Creates single-file `dist/cli.mjs` from `src/entrypoints/cli.tsx`
- Features:
  - `src-resolver` plugin for `src/` imports
  - `stub-missing` plugin for Anthropic-internal modules
  - Extensive `external` list (native addons, cloud SDKs, OTel exporters)
  - `define` replacements for `MACRO.*` and `USER_TYPE=external`
  - Post-build lodash-es ESM fix
  - Metafile output for bundle analysis

---

## Observability

### OpenTelemetry (6 packages)
- Traces, metrics, logs
- Instrumentation for API calls, tool execution, query loop

### Sentry
- Error tracking and reporting

### Pino Logging
- Structured JSON logging

### Prometheus Metrics
- Request counts, latencies, error rates
- Active sessions, streaming duration
- Token usage (input/output by model)
- Node.js runtime health (heap, event loop lag, GC)

---

## Database

### Drizzle ORM
- Dual dialect: SQLite (local) + PostgreSQL (production)
- Auto-detects from `DATABASE_URL`
- SQLite default: `~/.claude/web/claude-code.db`
- Migrations in `src/server/db/migrations/`

### Commands
```bash
bun run db:generate   # Generate migrations
bun run db:migrate    # Run migrations
bun run db:seed       # Seed database
bun run db:reset      # Reset database
bun run db:studio     # Drizzle Studio UI
```

---

## AI Providers

| Provider | Configuration |
|---|---|
| **Anthropic Direct** | `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL` |
| **AWS Bedrock** | `ANTHROPIC_BEDROCK_ENABLED`, region, base URL |
| **Google Vertex AI** | `ANTHROPIC_VERTEX_ENABLED`, region, project ID |
| **Azure Foundry** | `ANTHROPIC_FOUNDRY_ENABLED`, endpoint, subscription |

### Model Overrides
Per-tier model config (Opus, Sonnet, Haiku) with custom name/description labels, custom model option, and sub-agent model selection.

---

## Authentication Flow

4-step auth resolution chain:
1. **Provider selection** — Anthropic, Bedrock, Vertex, Foundry
2. **OAuth vs API key decision** — Check for API key first, fall back to OAuth
3. **API key resolution** — Env var, keychain, MDM, managed enterprise
4. **Client construction** — Build API client with resolved credentials

### Auth Variables
- `ANTHROPIC_API_KEY` — Direct API key
- `ANTHROPIC_AUTH_TOKEN` — OAuth token
- `ANTHROPIC_BASE_URL` — Custom API endpoint
- `ANTHROPIC_CUSTOM_HEADERS` — Additional headers
