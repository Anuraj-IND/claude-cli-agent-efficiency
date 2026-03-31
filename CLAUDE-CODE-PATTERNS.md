# Claude Code — Coding Patterns & Conventions

> ⚠️ READ-ONLY — Do NOT modify this file. This is a pre-built reference document.

> Coding conventions, patterns, anti-patterns, and best practices for working with the Claude Code codebase.

## Code Style

### Formatting (Biome)

| Setting | Value |
|---|---|
| Indent | Tabs |
| Indent width | 2 spaces |
| Line width | 100 characters |
| Quotes | Single quotes |
| Semicolons | As needed |
| Trailing commas | All |
| Organize imports | Enabled |

### Linter Rules

| Rule | Setting |
|---|---|
| `noExcessiveCognitiveComplexity` | warn |
| `noUnusedImports` | warn |
| `noUnusedVariables` | warn |
| `noNonNullAssertion` | **off** (allowed) |
| `noExplicitAny` | **off** (allowed) |
| `useImportType` | warn (prefer `import type`) |

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Files (components) | PascalCase.tsx | `ChatWindow.tsx` |
| Files (utils) | camelCase.ts | `messageQueueManager.ts` |
| Files (tools) | PascalCaseTool/ | `BashTool/` |
| Files (commands) | kebab-case/ | `/help/` |
| Components | PascalCase | `function ChatWindow()` |
| Hooks | camelCase, `use` prefix | `useChat()` |
| Types/Interfaces | PascalCase | `interface AppState` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Variables | camelCase | `sessionId` |
| Functions | camelCase | `submitMessage()` |
| Private fields | `_` prefix or `#` | `_internalState` |
| Test files | `*.test.ts` | `tools.test.ts` |

---

## Import Practices

### ES Modules with .js Extensions

Source files use `.js` extensions in imports (ESM convention), even though files are `.ts`/`.tsx`:

```typescript
// Correct
import { query } from './query.js';
import type { Message } from './types/message.js';

// The bundler resolves these to .ts/.tsx files
```

### Import Type vs Import

```typescript
// Prefer import type for types
import type { AppState } from './state/AppStateStore.js';

// Use regular import for values
import { buildTool } from './Tool.js';
```

### Lazy Loading

```typescript
// Dynamic import for heavy components
const HeavyComponent = await import('./HeavyComponent.js');

// Conditional require for feature-gated modules
if (feature('FLAG')) {
  const { featureModule } = require('./FeatureModule.js');
}
```

### Barrel Exports

```typescript
// index.ts — barrel export
export { ToolA } from './ToolA.js';
export { ToolB } from './ToolB.js';
export type { ToolType } from './types.js';
```

---

## TypeScript Patterns

### Discriminated Unions

```typescript
type Message =
  | { type: 'assistant'; content: string }
  | { type: 'user'; content: string }
  | { type: 'system'; content: string }
  | { type: 'tool_use'; name: string; input: unknown }
  | { type: 'tool_result'; tool_use_id: string; content: string };

// Type narrowing
function handleMessage(msg: Message) {
  switch (msg.type) {
    case 'assistant':
      // msg is { type: 'assistant'; content: string }
      break;
    case 'tool_use':
      // msg is { type: 'tool_use'; name: string; input: unknown }
      break;
  }
}
```

### Branded Types

```typescript
// Typed ID wrappers
type SessionId = string & { __brand: 'SessionId' };
type AgentId = string & { __brand: 'AgentId' };

// Factory functions
function createSessionId(id: string): SessionId {
  return id as SessionId;
}
```

### DeepImmutable

```typescript
// All state is wrapped in DeepImmutable
type ImmutableState = DeepImmutable<AppState>;

// Prevents accidental mutations
```

### Generics

```typescript
interface Tool<Input, Output, Progress> {
  call(input: Input, context: Context): Promise<Output>;
  // ...
}
```

---

## State Management

### Global State (bootstrap/state.ts)

```typescript
// Get state
const sessionId = getState().sessionId;

// Set state
setState({ sessionId: newId });

// DO NOT add more fields here — use AppState store instead
```

### AppState Store

```typescript
// Get state
const state = getState();

// Update state (immutable)
setState(prev => ({
  ...prev,
  conversations: [...prev.conversations, newConversation],
}));

// Subscribe to changes
subscribe(state => {
  console.log('State changed:', state);
});
```

---

## Feature Flags

### Build-time Flags

```typescript
import { feature } from 'bun:bundle';

if (feature('PROACTIVE')) {
  // This code is eliminated at build time if flag is false
}
```

### Runtime Gates

```typescript
// GrowthBook
if (growthbook.isOn('feature_name')) {
  // ...
}

// Environment checks
if (USER_TYPE === 'ant') {
  // Internal-only code
}

if (NODE_ENV === 'test') {
  // Test-only code
}
```

---

## Error Handling

### Fail-Open Philosophy

```typescript
// Good: Graceful degradation
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  logError(error);
  return fallbackValue;  // Don't crash
}

// Bad: Letting errors propagate
const result = await riskyOperation();  // May crash the app
```

### Ring Buffer for Errors

```typescript
// In-memory error ring buffer (max 100)
const errorBuffer = new RingBuffer<Error>(100);
errorBuffer.push(error);
```

### Circuit Breakers

```typescript
// Auto-compact circuit breaker
if (compactFailures > MAX_FAILURES) {
  disableAutoCompact();
  logWarning('Auto-compact disabled due to repeated failures');
}
```

---

## Performance Patterns

### Memoization

```typescript
import memoize from 'lodash-es/memoize';

const expensiveComputation = memoize((input: string) => {
  // Expensive operation
  return result;
});
```

### Deferred Work

```typescript
// Fire-and-forget for non-critical async work
void nonCriticalOperation();

// Deferred prefetches after first render
requestIdleCallback(() => {
  prefetchData();
});
```

### Startup Profiling

```typescript
// Checkpoint profiling
const checkpoint = startCheckpoint('module-load');
// ... module loading ...
endCheckpoint(checkpoint);
```

### Lazy Component Loading

```typescript
// In React components
const LazySettings = lazy(() => import('./Settings.js'));

// In UI
<Suspense fallback={<LoadingSpinner />}>
  <LazySettings />
</Suspense>
```

---

## React/Ink Patterns

### Component Structure

```typescript
import { Box, Text } from 'ink';
import React from 'react';

interface MyComponentProps {
  title: string;
  children?: React.ReactNode;
}

export function MyComponent({ title, children }: MyComponentProps) {
  return (
    <Box flexDirection="column">
      <Text bold>{title}</Text>
      {children}
    </Box>
  );
}
```

### Ink Hooks

```typescript
import { useInput, useApp, useStdin } from 'ink';

function MyInteractiveComponent() {
  const { exit } = useApp();
  const { stdin } = useStdin();

  useInput((input, key) => {
    if (key.escape) {
      exit();
    }
    if (input === 'q') {
      exit();
    }
  });

  return <Text>Press q or Escape to quit</Text>;
}
```

### State in Components

```typescript
// Use AppState via context
function MyComponent() {
  const settings = useAppState(s => s.settings);
  const updateSettings = useAppState(s => s.updateSettings);

  return <Text>Theme: {settings.theme}</Text>;
}
```

---

## Async Generator Pattern

### Query Loop

```typescript
async function* queryLoop(): AsyncGenerator<StreamEvent | Message> {
  while (true) {
    // 1. Prefetch skills
    const skills = await prefetchSkills();

    // 2. Compact if needed
    if (shouldCompact()) {
      yield await compact();
    }

    // 3. Stream API response
    const stream = await api.stream(messages);
    for await (const event of stream) {
      yield event;
    }

    // 4. Execute tool calls
    const toolResults = await executeTools(toolCalls);
    yield toolResults;
  }
}
```

---

## Tool Definition Pattern

```typescript
import { buildTool } from './Tool.js';
import { z } from 'zod';

export const myTool = buildTool({
  name: 'MyTool',
  description: 'Does something useful',
  inputSchema: z.object({
    param: z.string().describe('Parameter description'),
  }),
  call: async ({ param }, context) => {
    try {
      const result = await doSomething(param);
      return {
        content: [{ type: 'text', text: result }],
      };
    } catch (error) {
      return {
        content: [{ type: 'text', text: error.message }],
        isError: true,
      };
    }
  },
  // Optional
  isReadOnly: true,
  checkPermissions: () => ({ type: 'autoApprove' }),
});
```

---

## Command Definition Pattern

### Prompt Command (Skill)

```markdown
---
name: my-skill
description: Does something useful
type: prompt
---

Instructions for the AI...
```

### Local Command

```typescript
export const myCommand: LocalCommand = {
  name: 'my-command',
  description: 'Does something',
  type: 'local',
  availability: 'claude-ai',
  enabled: true,
  loadedFrom: 'bundled',
  action: async (args, context) => {
    return 'Result';
  },
};
```

### Local JSX Command

```typescript
export const myCommand: LocalJSXCommand = {
  name: 'my-command',
  description: 'Interactive dialog',
  type: 'local-jsx',
  availability: 'claude-ai',
  enabled: true,
  loadedFrom: 'bundled',
  component: MyComponent,
};
```

---

## Anti-Patterns

### DON'T Do This

```typescript
// Don't add to global state
STATE.newField = value;  // BAD

// Don't use synchronous imports for heavy modules
import { HeavyComponent } from './HeavyComponent.js';  // BAD

// Don't let errors crash the app
const result = await riskyOperation();  // BAD (no try/catch)

// Don't mutate state directly
state.conversations.push(newConv);  // BAD

// Don't use any when a type is available
function process(data: any) {}  // BAD (prefer proper type)
```

### DO This Instead

```typescript
// Use AppState store
setState(prev => ({ ...prev, newField: value }));

// Lazy load heavy modules
const HeavyComponent = await import('./HeavyComponent.js');

// Handle errors gracefully
try {
  const result = await riskyOperation();
} catch (error) {
  logError(error);
  return fallback;
}

// Immutable state updates
setState(prev => ({
  ...prev,
  conversations: [...prev.conversations, newConv],
}));

// Use proper types
function process(data: ProcessableData) {}
```

---

## Testing Patterns

### Unit Test

```typescript
import { describe, it, expect, beforeEach } from 'vitest';

describe('myFunction', () => {
  beforeEach(() => {
    // Reset state
    cache.clear();
  });

  it('should return expected result', () => {
    const result = myFunction('input');
    expect(result).toBe('expected');
  });

  it('should handle edge case', () => {
    expect(() => myFunction('')).toThrow();
  });
});
```

### Smoke Test

```typescript
describe('module', () => {
  it('should export required fields', () => {
    const exports = getExports();
    expect(exports).toHaveProperty('name');
    expect(exports).toHaveProperty('description');
  });
});
```

### Integration Test

```typescript
describe('protocol', () => {
  it('should handle roundtrip', async () => {
    const server = createTestServer();
    const client = createTestClient(server);

    const result = await client.call('method', { param: 'value' });
    expect(result).toEqual({ output: 'expected' });
  });
});
```

---

## File Organization

### Tool Directory

```
src/tools/MyTool/
├── index.ts          # Export the tool
├── MyTool.ts         # Tool definition
├── schema.ts         # Input/output schemas
├── handler.ts        # Core logic
└── __tests__/
    └── MyTool.test.ts
```

### Command Directory

```
src/commands/my-command/
├── index.ts          # Export the command
├── MyCommand.ts      # Command definition
└── __tests__/
    └── MyCommand.test.ts
```

### Component Directory

```
src/components/MyComponent/
├── index.ts          # Barrel export
├── MyComponent.tsx   # Main component
├── MyComponent.styles.ts  # Styles (if needed)
└── __tests__/
    └── MyComponent.test.tsx
```

---

## Validation Commands

After making changes, always run:

```bash
bun run check      # Lint + typecheck
bun run test       # Run tests
bun run build      # Build bundle
```
