# Claude Code — Command System Reference

> ⚠️ READ-ONLY — Do NOT modify this file. This is a pre-built reference document.

> Complete reference for the slash command system: types, patterns, and all built-in commands.

## Command Types

### Three Command Types

| Type | Description | Use Case | Example |
|---|---|---|---|
| `prompt` | Expands to text sent to the model | Skills, custom prompts | `/skill`, `.claude/commands/*.md` |
| `local` | Executes locally, returns text | Info, status, help | `/help`, `/status`, `/cost` |
| `local-jsx` | Renders Ink UI components | Interactive dialogs | `/config`, `/model`, `/theme` |

### Command Type Schema

```typescript
interface BaseCommand {
  name: string;
  description: string;
  availability: 'claude-ai' | 'console';
  enabled: boolean;
  loadedFrom: 'commands_DEPRECATED' | 'skills' | 'plugin' | 'managed' | 'bundled' | 'mcp';
}

interface PromptCommand extends BaseCommand {
  type: 'prompt';
  template: string;       // Markdown template sent to the model
  hooks?: CommandHooks;
}

interface LocalCommand extends BaseCommand {
  type: 'local';
  action: (args: string[], context: CommandContext) => Promise<string>;
}

interface LocalJSXCommand extends BaseCommand {
  type: 'local-jsx';
  component: React.FC<CommandProps>;
}

type Command = PromptCommand | LocalCommand | LocalJSXCommand;
```

### Command Availability

| Availability | Description |
|---|---|
| `claude-ai` | Available in the AI conversation (most commands) |
| `console` | Available in the console/REPL only |

### Command loadedFrom Values

| Value | Description |
|---|---|
| `commands_DEPRECATED` | Legacy built-in commands |
| `skills` | Skill-based commands |
| `plugin` | Plugin-provided commands |
| `managed` | Managed/enterprise commands |
| `bundled` | Bundled commands (shipped with the app) |
| `mcp` | MCP server-provided commands |

---

## Command Registry

### Location: `src/commands.ts`

```typescript
// Get all available commands
function getCommands(options: {
  availability: 'claude-ai' | 'console';
}): Command[];

// Get skills as tool-invocable commands
function getSkillToolCommands(): SkillCommand[];

// Get slash commands for tool skills
function getSlashCommandToolSkills(): SlashCommandSkill[];

// Built-in command array
function COMMANDS(): Command[];

// Internal-only commands (ant users only)
const INTERNAL_ONLY_COMMANDS: Command[];

// Safe commands for remote mode
const REMOTE_SAFE_COMMANDS: string[];

// Safe commands for bridge mode
const BRIDGE_SAFE_COMMANDS: string[];
```

### Feature-Flagged Commands

| Command | Flag |
|---|---|
| `/proactive` | `PROACTIVE` |
| `/brief` | `BRIEF` |
| `/assistant` | `ASSISTANT` |
| `/bridge` | `BRIDGE_MODE` |
| `/remoteControlServer` | `REMOTE_CONTROL` |
| `/voice` | `VOICE_MODE` |
| `/force-snip` | `HISTORY_SNIP` |
| `/workflows` | `WORKFLOW` |
| `/ultraplan` | `ULTRAPLAN` |
| `/torch` | `TORCH` |
| `/peers` | `PEERS` |
| `/fork` | `FORK` |
| `/buddy` | `BUDDY` |
| `/subscribe-pr` | `SUBSCRIBE_PR` |

---

## All Built-in Commands

### Session Management

| Command | Type | Description |
|---|---|---|
| `/clear` | local | Clear the conversation |
| `/compact` | local | Compact the conversation context |
| `/resume` | local-jsx | Resume a previous session |
| `/session` | local-jsx | Session picker and management |
| `/rename` | local | Rename the current session |
| `/rewind` | local-jsx | Rewind to a previous point in conversation |
| `/export` | local-jsx | Export conversation |
| `/copy` | local | Copy conversation text |

### Configuration

| Command | Type | Description |
|---|---|---|
| `/config` | local-jsx | Open settings dialog |
| `/model` | local-jsx | Change the AI model |
| `/theme` | local-jsx | Change the terminal theme |
| `/permissions` | local-jsx | Configure permission mode |
| `/hooks` | local-jsx | Manage hooks |
| `/memory` | local-jsx | Manage session memory |
| `/skills` | local-jsx | Manage skills |
| `/plugins` | local-jsx | Manage plugins |

### Git/PR Operations

| Command | Type | Description |
|---|---|---|
| `/diff` | local | Show git diff |
| `/branch` | local | Show current branch |
| `/commit` | local | Create a git commit |
| `/commit-push-pr` | local | Commit, push, and create PR |
| `/review` | local | Review code changes |
| `/pr_comments` | local | Fetch PR comments |

### MCP Management

| Command | Type | Description |
|---|---|---|
| `/mcp` | local-jsx | Manage MCP servers (add, list, remove) |

### Authentication

| Command | Type | Description |
|---|---|---|
| `/login` | local-jsx | Authenticate with Anthropic |
| `/logout` | local | Log out |
| `/install-github-app` | local | Install GitHub app integration |
| `/install-slack-app` | local | Install Slack app integration |

### Information & Diagnostics

| Command | Type | Description |
|---|---|---|
| `/help` | local | Show help information |
| `/status` | local | Show current status |
| `/stats` | local | Show usage statistics |
| `/cost` | local | Show session cost |
| `/usage` | local | Show token usage |
| `/doctor` | local-jsx | Run diagnostics |
| `/context` | local | Show current context |

### UI Customization

| Command | Type | Description |
|---|---|---|
| `/output-style` | local-jsx | Change output style |
| `/keybindings` | local-jsx | View/customize keybindings |
| `/vim` | local | Toggle vim mode |
| `/statusline` | local-jsx | Configure status line |
| `/stickers` | local-jsx | Toggle sticker animations |
| `/color` | local-jsx | Configure color settings |

### Feature-Specific

| Command | Type | Description |
|---|---|---|
| `/desktop` | local-jsx | Desktop app settings |
| `/mobile` | local | Mobile mode toggle |
| `/chrome` | local | Chrome extension settings |
| `/ide` | local | IDE integration settings |
| `/teleport` | local | Teleport feature |
| `/bridge` | local | Bridge mode settings |
| `/btw` | local | "By the way" feature |
| `/plan` | local | Enter plan mode |
| `/fast` | local | Toggle fast mode |
| `/passes` | local | View passes/credits |
| `/effort` | local | Set effort level |
| `/feedback` | local-jsx | Submit feedback |

### Internal/Debug (Ant Users Only)

| Command | Type | Description |
|---|---|---|
| `/ant-trace` | local | Ant trace logging |
| `/perf-issue` | local | Performance issue diagnostics |
| `/mock-limits` | local | Mock API limits for testing |
| `/reset-limits` | local | Reset API limits |
| `/good-claude` | local | Good Claude scoring |
| `/issue` | local | Create an issue |
| `/bughunter` | local | Bug hunting mode |
| `/ctx_viz` | local | Context visualization |
| `/debug-tool-call` | local | Debug tool call |
| `/summary` | local | Generate summary |
| `/share` | local | Share conversation |

### Ant-Internal Only Commands

| Command | Description |
|---|---|
| `/backfill-sessions` | Backfill session data |
| `/break-cache` | Break cache |
| `/bughunter` | Bug hunting mode |
| `/teleport` | Teleport feature |

---

## Command Definition Patterns

### Prompt Command (Skill)

```markdown
---
name: my-skill
description: Does something useful
type: prompt
---

You are a helpful assistant that does X. When the user asks you to do Y:

1. First, check the current state
2. Then, perform the action
3. Finally, confirm completion

Use the following tools: Bash, Read, Write, Edit.
```

### Local Command

```typescript
import type { LocalCommand } from 'src/types/command';

const myCommand: LocalCommand = {
  name: 'my-command',
  description: 'Does something useful',
  type: 'local',
  availability: 'claude-ai',
  enabled: true,
  loadedFrom: 'bundled',
  action: async (args, context) => {
    // Implementation
    return `Result: ${args.join(' ')}`;
  },
};
```

### Local JSX Command

```typescript
import type { LocalJSXCommand } from 'src/types/command';

const myCommand: LocalJSXCommand = {
  name: 'my-command',
  description: 'Interactive dialog',
  type: 'local-jsx',
  availability: 'claude-ai',
  enabled: true,
  loadedFrom: 'bundled',
  component: MyCommandComponent,
};

function MyCommandComponent(props: CommandProps) {
  const [value, setValue] = useState('');

  return (
    <Box flexDirection="column">
      <Text>Enter a value:</Text>
      <TextInput value={value} onChange={setValue} />
      <Text>Press Enter to confirm</Text>
    </Box>
  );
}
```

---

## Command Context

```typescript
interface CommandContext {
  appState: AppState;
  session: Session;
  settings: Settings;
  // Access to tools, services, etc.
}
```

---

## Command Hooks

```typescript
interface CommandHooks {
  before?: (context: HookContext) => Promise<void>;
  after?: (context: HookContext) => Promise<void>;
  onError?: (error: Error, context: HookContext) => Promise<void>;
}
```

---

## Remote-Safe Commands

Commands allowed in remote mode (`REMOTE_SAFE_COMMANDS`):
- `/help`, `/status`, `/stats`
- `/config`, `/model`
- `/clear`, `/compact`
- `/permissions`
- Git commands (read-only)

## Bridge-Safe Commands

Commands allowed in bridge mode (`BRIDGE_SAFE_COMMANDS`):
- Same as remote-safe, plus:
- `/mcp` (read-only operations)
- `/context`

---

## Command Loading Order

1. **Bundled commands** — Shipped with the application
2. **Skills** — From `.claude/skills/` directory
3. **Plugins** — From installed plugins
4. **Managed** — From enterprise management
5. **MCP** — From connected MCP servers
6. **User commands** — From `.claude/commands/` directory

Commands are deduplicated by name, with later sources overriding earlier ones.

---

## Command Testing

### Smoke Test Pattern

```typescript
import { getCommands } from 'src/commands';

describe('commands', () => {
  it('should have core commands', () => {
    const commands = getCommands({ availability: 'claude-ai' });
    expect(commands.length).toBeGreaterThan(0);

    const names = commands.map(c => c.name);
    expect(names).toContain('help');
    expect(names).toContain('config');

    // Every command should have required fields
    for (const cmd of commands) {
      expect(cmd.name).toBeDefined();
      expect(cmd.description).toBeDefined();
      expect(typeof cmd.name).toBe('string');
      expect(typeof cmd.description).toBe('string');
    }
  });
});
```
