# Claude Code — Tool System Reference

> ⚠️ READ-ONLY — Do NOT modify this file. This is a pre-built reference document.

> Complete reference for the tool system: schema, permissions, patterns, and all built-in tools.

## Tool Interface Schema

### Core Interface

```typescript
interface Tool<Input, Output, Progress> {
  // Required
  name: string;
  description: string;
  inputSchema: z.ZodType<Input>;
  call(input: Input, context: ToolUseContext): Promise<Output>;
  prompt(input: Input): string;
  userFacingName(): string;
  renderToolUseMessage(input: Input): ReactElement;
  checkPermissions(input: Input, context: ToolUseContext): PermissionDecision;
  mapToolResultToToolResultBlockParam(result: Output): ToolResultBlockParam;

  // Optional (30+)
  isEnabled?(context: ToolUseContext): boolean;
  isConcurrencySafe?: boolean;
  isReadOnly?: boolean;
  isDestructive?: boolean;
  interruptBehavior?: InterruptBehavior;
  isSearchOrReadCommand?: boolean;
  isOpenWorld?: boolean;
  requiresUserInteraction?: boolean;
  shouldDefer?: boolean;
  alwaysLoad?: boolean;
  backfillObservableInput?(input: Input): Input;
  validateInput?(input: Input): ValidationError | null;
  preparePermissionMatcher?(input: Input): PermissionMatcher;
  getToolUseSummary?(input: Input): string;
  getActivityDescription?(input: Input): string;
  toAutoClassifierInput?(input: Input): AutoClassInput;
  renderToolResultMessage?(result: Output): ReactElement;
  extractSearchText?(result: Output): string;
  isResultTruncated?(result: Output): boolean;
  renderToolUseTag?(input: Input): ReactElement;
  renderToolUseProgressMessage?(progress: Progress): ReactElement;
  renderToolUseQueuedMessage?(): ReactElement;
  renderToolUseRejectedMessage?(reason: string): ReactElement;
  renderToolUseErrorMessage?(error: Error): ReactElement;
  renderGroupedToolUse?(inputs: Input[]): ReactElement;
}
```

### ToolUseContext

```typescript
interface ToolUseContext {
  appState: AppState;
  mcpClients: McpClient[];
  abortController: AbortController;
  notifications: NotificationManager;
  fileHistory: FileHistory;
  attribution: AttributionTracker;
  sessionMemory: SessionMemory;
  // ... many more fields
}
```

### Permission Decision

```typescript
type PermissionDecision =
  | { type: 'autoApprove' }
  | { type: 'askUser' }
  | { type: 'deny' }
  | { type: 'block' };
```

---

## Tool Definition Pattern

### Minimal Tool

```typescript
import { buildTool } from 'src/Tool';
import { z } from 'zod';

const myTool = buildTool({
  name: 'MyTool',
  description: 'Does something useful for the AI assistant',
  inputSchema: z.object({
    param1: z.string().describe('What this param does'),
    param2: z.number().optional().describe('Optional param'),
  }),
  call: async ({ param1, param2 }, context) => {
    // Implementation
    return {
      content: [
        { type: 'text', text: `Result: ${param1}` },
      ],
    };
  },
});
```

### Full Tool with All Options

```typescript
const myTool = buildTool({
  name: 'MyTool',
  description: 'Does something useful',
  inputSchema: z.object({
    path: z.string().describe('File path to operate on'),
    content: z.string().describe('Content to write'),
  }),
  call: async ({ path, content }, context) => {
    try {
      await fs.writeFile(path, content);
      return {
        content: [
          { type: 'text', text: `Successfully wrote to ${path}` },
        ],
      };
    } catch (error) {
      return {
        content: [
          { type: 'text', text: `Error: ${error.message}` },
        ],
        isError: true,
      };
    }
  },
  prompt: ({ path }) => `Write content to ${path}`,
  userFacingName: () => 'Write File',
  isReadOnly: false,
  isDestructive: true,
  isConcurrencySafe: false,
  checkPermissions: ({ path }, context) => {
    if (path.startsWith('/etc/')) {
      return { type: 'deny' };
    }
    return { type: 'askUser' };
  },
  renderToolUseMessage: ({ path }) => {
    return <Text>Writing to {path}</Text>;
  },
  getToolUseSummary: ({ path }) => `Wrote file: ${path}`,
  getActivityDescription: ({ path }) => `Writing to ${path}`,
});
```

---

## All Built-in Tools

### File System Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **Read** | `src/tools/FileReadTool/` | Yes | Read file contents with line numbers |
| **Write** | `src/tools/FileWriteTool/` | No | Write content to a file (creates or overwrites) |
| **Edit** | `src/tools/FileEditTool/` | No | Apply SEARCH/REPLACE blocks to a file |
| **Glob** | `src/tools/GlobTool/` | Yes | Find files matching a glob pattern |
| **Grep** | `src/tools/GrepTool/` | Yes | Search file contents with regex |
| **NotebookEdit** | `src/tools/NotebookEditTool/` | No | Edit Jupyter notebook cells |
| **FileRead** | `src/tools/FileReadTool/` | Yes | Alternative file read implementation |

### Shell/Execution Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **Bash** | `src/tools/BashTool/` | No | Execute bash commands in a sandbox |
| **PowerShell** | `src/tools/PowerShellTool/` | No | Execute PowerShell commands |
| **TerminalCapture** | `src/tools/TerminalCaptureTool/` | Yes | Capture terminal output |

### Web Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **WebFetch** | `src/tools/WebFetchTool/` | Yes | Fetch and read web page content |
| **WebSearch** | `src/tools/WebSearchTool/` | Yes | Search the web using Exa AI |

### Agent/Orchestration Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **Agent** | `src/tools/AgentTool/` | No | Spawn a sub-agent with a prompt |
| **TaskCreate** | `src/tools/TaskCreateTool/` | No | Create a background task |
| **TaskGet** | `src/tools/TaskGetTool/` | Yes | Get task status and output |
| **TaskUpdate** | `src/tools/TaskUpdateTool/` | No | Update a running task |
| **TaskList** | `src/tools/TaskListTool/` | Yes | List all tasks |
| **TaskOutput** | `src/tools/TaskOutputTool/` | Yes | Get task output stream |
| **TaskStop** | `src/tools/TaskStopTool/` | No | Stop a running task |
| **TeamCreate** | `src/tools/TeamCreateTool/` | No | Create a multi-agent team |
| **TeamDelete** | `src/tools/TeamDeleteTool/` | No | Delete a team |
| **SendMessage** | `src/tools/SendMessageTool/` | No | Send message to a task/agent |

### Planning Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **EnterPlanMode** | `src/tools/EnterPlanModeTool/` | No | Enter planning mode |
| **ExitPlanMode** | `src/tools/ExitPlanModeTool/` | No | Exit planning mode |
| **TodoWrite** | `src/tools/TodoWriteTool/` | No | Write/update todo list |
| **Brief** | `src/tools/BriefTool/` | No | Create a task brief |
| **VerifyPlan** | `src/tools/VerifyPlanTool/` | Yes | Verify a plan |

### MCP Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **ListMcpResources** | `src/tools/ListMcpResourcesTool/` | Yes | List MCP server resources |
| **ReadMcpResource** | `src/tools/ReadMcpResourceTool/` | Yes | Read an MCP resource |
| **McpAuth** | `src/tools/McpAuthTool/` | No | Authenticate with MCP server |

### Special Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **REPL** | `src/tools/REPLTool/` | No | Enter REPL mode |
| **Skill** | `src/tools/SkillTool/` | No | Load and use a skill |
| **Config** | `src/tools/ConfigTool/` | No | Modify configuration |
| **Sleep** | `src/tools/SleepTool/` | Yes | Pause execution |
| **ScheduleCron** | `src/tools/ScheduleCronTool/` | No | Schedule a cron job |
| **LSP** | `src/tools/LSPTool/` | Yes | Language server protocol operations |

### Advanced Tools

| Tool | Location | Read-Only | Description |
|---|---|---|---|
| **ToolSearch** | `src/tools/ToolSearchTool/` | Yes | Search for deferred tool loading |
| **SyntheticOutput** | `src/tools/SyntheticOutputTool/` | Yes | Structured output generation |
| **Workflow** | `src/tools/WorkflowTool/` | No | Execute workflow scripts |

---

## Tool Permissions Schema

### Permission Modes

| Mode | Auto-approves | Asks for |
|---|---|---|
| `default` | Read-only tools | Destructive operations |
| `acceptEdits` | File edits | Everything else |
| `dangerous` | Everything except blocked | Nothing (unless blocked) |
| `fullAuto` | Everything | Nothing |

### Permission Rule Schema

```typescript
interface PermissionRule {
  tool: string;           // Tool name or wildcard
  command?: string;       // Command pattern (for Bash tool)
  path?: string;          // File path pattern
  decision: 'allow' | 'deny' | 'ask';
}
```

### Wildcard Patterns

```
*         — Match any single path segment
**        — Match any path (recursive)
*.ts      — Match all TypeScript files
src/**    — Match everything under src/
```

---

## Tool Input Schemas

### Bash Tool

```typescript
inputSchema: z.object({
  command: z.string().describe('The bash command to execute'),
  description: z.string().optional().describe('Why you are running this command'),
  run_in_background: z.boolean().optional().describe('Run as background task'),
})
```

### Read Tool

```typescript
inputSchema: z.object({
  file_path: z.string().describe('Absolute path to the file to read'),
  offset: z.number().optional().describe('Line number to start reading from (1-indexed)'),
  limit: z.number().optional().describe('Maximum number of lines to read'),
})
```

### Edit Tool

```typescript
inputSchema: z.object({
  file_path: z.string().describe('Absolute path to the file to edit'),
  old_string: z.string().describe('Text to search for (must be unique)'),
  new_string: z.string().describe('Text to replace with'),
  replace_all: z.boolean().optional().describe('Replace all occurrences'),
})
```

### Write Tool

```typescript
inputSchema: z.object({
  file_path: z.string().describe('Absolute path to the file to write'),
  content: z.string().describe('Content to write to the file'),
})
```

### Glob Tool

```typescript
inputSchema: z.object({
  pattern: z.string().describe('Glob pattern to match files'),
  path: z.string().optional().describe('Directory to search in'),
})
```

### Grep Tool

```typescript
inputSchema: z.object({
  pattern: z.string().describe('Regex pattern to search for'),
  path: z.string().optional().describe('File or directory to search in'),
  glob: z.string().optional().describe('File pattern to filter by'),
  output_mode: z.enum(['content', 'files_with_matches', 'count']).optional(),
  -C: z.number().optional().describe('Context lines'),
  -i: z.boolean().optional().describe('Case insensitive'),
})
```

### WebFetch Tool

```typescript
inputSchema: z.object({
  url: z.string().describe('URL to fetch'),
  max_length: z.number().optional().describe('Maximum content length'),
})
```

### WebSearch Tool

```typescript
inputSchema: z.object({
  query: z.string().describe('Search query'),
  num_results: z.number().optional().describe('Number of results to return'),
  live_crawl: z.enum(['fallback', 'preferred']).optional(),
  type: z.enum(['auto', 'fast', 'deep']).optional(),
})
```

### Agent Tool

```typescript
inputSchema: z.object({
  prompt: z.string().describe('What you want the agent to do'),
  model: z.string().optional().describe('Model to use'),
  permission_mode: z.enum(['default', 'acceptEdits', 'dangerous', 'fullAuto']).optional(),
  tools: z.array(z.string()).optional().describe('Tools available to the agent'),
  description: z.string().optional().describe('Short description of the task'),
})
```

### TodoWrite Tool

```typescript
inputSchema: z.object({
  todos: z.array(z.object({
    content: z.string().describe('Todo item text'),
    status: z.enum(['pending', 'in_progress', 'completed', 'cancelled']),
    priority: z.enum(['high', 'medium', 'low']).optional(),
  })),
})
```

---

## Tool Result Schema

### Success Result

```typescript
{
  content: [
    { type: 'text', text: 'Human-readable result' },
    { type: 'image', data: 'base64...', mimeType: 'image/png' },  // optional
  ],
  metadata?: { /* tool-specific metadata */ },
}
```

### Error Result

```typescript
{
  content: [
    { type: 'text', text: 'Error message' },
  ],
  isError: true,
}
```

### Truncated Result

```typescript
{
  content: [
    { type: 'text', text: 'Partial result...' },
  ],
  isTruncated: true,
  metadata: {
    truncatedBytes: 1024,
    totalBytes: 50000,
  },
}
```

---

## Tool Registry

### Location: `src/tools.ts`

```typescript
// Get all built-in tools
function getAllBaseTools(): Tool[];

// Get filtered tools by mode
function getTools(options: {
  simpleMode?: boolean;
  replMode?: boolean;
  denyRules?: PermissionRule[];
}): Tool[];

// Assemble tool pool (built-in + MCP)
function assembleToolPool(options: {
  builtInTools: Tool[];
  mcpTools: Tool[];
}): ToolPool;
```

### Feature-Flagged Tools

These tools are conditionally included based on feature flags:

| Tool | Flag |
|---|---|
| REPL | `REPL` |
| Sleep | `SLEEP` |
| Cron | `CRON` |
| RemoteTrigger | `REMOTE_TRIGGER` |
| Monitor | `MONITOR` |
| SendUserFile | `SEND_USER_FILE` |
| PushNotification | `PUSH_NOTIFICATION` |
| SubscribePR | `SUBSCRIBE_PR` |
| Tungsten | `TUNGSTEN` |
| VerifyPlan | `VERIFY_PLAN` |
| OverflowTest | `OVERFLOW_TEST` |
| CtxInspect | `CTX_INSPECT` |
| TerminalCapture | `TERMINAL_CAPTURE` |
| WebBrowser | `WEB_BROWSER` |
| Snip | `SNIP` |
| ListPeers | `LIST_PEERS` |
| Workflow | `WORKFLOW` |
| PowerShell | `POWER_SHELL` |

### Environment-Conditional Tools

| Tool | Condition |
|---|---|
| LSP | `ENABLE_LSP_TOOL` env var |
| Internal tools | `USER_TYPE === 'ant'` |
| Test tools | `NODE_ENV === 'test'` |

---

## Tool Testing

### Test Pattern

```typescript
import { describe, it, expect } from 'vitest';
import { myTool } from './MyTool';

describe('MyTool', () => {
  it('should have correct name', () => {
    expect(myTool.name).toBe('MyTool');
  });

  it('should validate input', () => {
    expect(() => myTool.inputSchema.parse({})).toThrow();
  });

  it('should call successfully', async () => {
    const result = await myTool.call(
      { param1: 'test' },
      mockContext
    );
    expect(result.content[0].text).toContain('expected');
  });
});
```

### Smoke Test Pattern

```typescript
import { getAllBaseTools } from 'src/tools';

describe('tools', () => {
  it('should have core tools', () => {
    const tools = getAllBaseTools();
    expect(tools.length).toBeGreaterThan(0);
    const names = tools.map(t => t.name);
    expect(names).toContain('Bash');
    expect(names).toContain('Read');
    expect(names).toContain('Write');
  });
});
```
