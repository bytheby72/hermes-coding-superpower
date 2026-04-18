# Claude Code Architecture Patterns

Architecture patterns and design insights from Claude Code CLI reverse engineering.

---

## 1. Agent System Architecture

### Core Loop Structure

```
User Input → Context Assembly → LLM Call → Tool Parse → Execute → Loop
```

### Key Components

| Component | Purpose | Hermes Equivalent |
|-----------|---------|-------------------|
| **System Prompt Assembly** | Dynamic prompt construction from skills, memory, context | `agent/prompt_builder.py` |
| **Context Compressor** | Auto-compress conversation history when near token limit | `agent/context_compressor.py` |
| **Tool Registry** | Central tool discovery and dispatch | `tools/registry.py` |
| **Stream Handler** | Process LLM streaming events (text/tool_use/usage) | `run_agent.py` |

### Sub-Agent Pattern (Swarm Mode)

```typescript
// Task decomposition with dependency management
interface Task {
  id: string
  subject: string
  description: string
  status: 'pending' | 'in_progress' | 'blocked' | 'done'
  blockedBy?: string[]  // Dependencies
  blocks?: string[]     // Downstream tasks
  owner?: string        // Assigned sub-agent
  activeForm?: string   // UX spinner text
}

// Sub-agent 认领任务模式
1. Coordinator 分析任务依赖图
2. 子 Agent 认领无依赖任务
3. 完成后自动解锁 blockedBy 任务
4. 递归直到所有任务完成
```

**Hermes 已有**: `delegate_task` 支持并行子代理，但缺少任务依赖管理。

---

## 2. Tool System Design

### Tool Interface Pattern

```typescript
interface Tool {
  name: string
  description: string
  inputSchema: JSONSchema
  handler: (args: object) => Promise<ToolResult>
}

interface ToolResult {
  content: ContentBlock[]
  isError?: boolean
}
```

### Built-in Tool Categories

| Category | Tools | Hermes Status |
|----------|-------|---------------|
| **File Operations** | Read, Write, Edit, Search | ✅ `read_file`, `write_file`, `patch`, `search_files` |
| **Search & Navigation** | Glob, Grep (ripgrep embedded) | ✅ `search_files` (ripgrep-backed) |
| **Shell Execution** | Bash (with permission gating) | ✅ `terminal` (with approval system) |
| **Task Management** | Todo create/update/list | ✅ `todo` tool |
| **Web** | Fetch, Search | ✅ `web_search`, `browser_*` tools |

### Tool Discovery Pattern

```typescript
// Semantic tool matching when user references tools by description
function ToolSearch(query: string, allTools: Tool[]): Tool[] {
  // Match against tool name + description using embeddings or keyword search
  // Default limit: 250 results (token budget consideration)
  return allTools
    .filter(t => matches(query, t.name, t.description))
    .slice(0, 250)
}
```

**Hermes 已有**: Tool registry with `check_fn` for availability, but no semantic search.

---

## 3. Safety & Permission Model

### Three-Tier Gating

```
Tier 1: Auto-approve (Read, Glob, Todo read)
Tier 2: Auto-approve with logging (Write within project, Search)
Tier 3: Require user confirmation (Bash, Write outside project, Network)
```

### Sandbox Pattern (macOS)

```typescript
// sandbox-exec profile for command isolation
function getSandboxConfig(command: string): SandboxConfig {
  // Read commands skip sandbox
  if (isReadCommand(command)) return { enabled: false }
  
  // Dangerous commands wrapped in seatbelt profile
  return {
    enabled: true,
    profile: `(version 1
      (deny default)
      (allow file-read-metadata (regex #"/.*"))
      (allow file-read-data (subpath "/allowed/path"))
      (allow process-exec (literal "/bin/bash"))
    )`
  }
}
```

### Plan Mode

```typescript
// Execute without side effects - analysis only
interface PlanMode {
  enabled: boolean
  // In plan mode:
  // - No file writes
  // - No bash execution
  // - Returns proposed changes as diff
}
```

**Hermes 已有**: `terminal_tool.py` has approval system, `approval.py` for dangerous command detection.

---

## 4. Token Budget Management

### Context Window Strategy

```typescript
interface TokenBudget {
  modelLimit: number      // e.g., 200000 for Claude
  reservedForResponse: number  // e.g., 4000
  availableForContext: number  // modelLimit - reserved
  
  // Compaction triggers
  compactionThreshold: number  // e.g., 80% of available
}

// When threshold exceeded:
1. Summarize oldest messages
2. Keep tool results compact
3. Preserve system prompt and recent turns
```

### Compaction Pattern

```typescript
async function compactConversation(history: Message[]): Promise<Message[]> {
  // Keep: system prompt, last 2 turns, tool calls
  // Summarize: older turns using auxiliary LLM call
  const summary = await summarize(history.slice(0, -4))
  return [summary, ...history.slice(-4)]
}
```

**Hermes 已有**: `agent/context_compressor.py` and `agent/prompt_caching.py`.

---

## 5. Streaming & Event Handling

### Stream Event Types

```typescript
type StreamEvent =
  | { type: 'content_block_delta'; delta: { text: string } }
  | { type: 'content_block_start'; block: { type: 'tool_use'; id: string; name: string } }
  | { type: 'content_block_stop' }
  | { type: 'message_delta'; delta: { stop_reason: string } }
  | { type: 'message_stop'; usage: { input_tokens: number; output_tokens: number } }
```

### Event Processor Pattern

```typescript
async function* processStreamEvents(stream: AsyncIterable<StreamEvent>) {
  let currentToolCall: ToolCall | null = null
  
  for await (const event of stream) {
    switch (event.type) {
      case 'content_block_delta':
        yield { type: 'text', content: event.delta.text }
        break
      case 'content_block_start':
        if (event.block.type === 'tool_use') {
          currentToolCall = { id: event.block.id, name: event.block.name, args: '' }
        }
        break
      case 'content_block_stop':
        if (currentToolCall) {
          yield { type: 'tool_call', call: currentToolCall }
          currentToolCall = null
        }
        break
    }
  }
}
```

**Hermes 已有**: `run_agent.py` handles tool calls from response, but streaming is provider-dependent.

---

## 6. MCP Protocol Integration

### MCP Tool Discovery

```typescript
// Model Context Protocol for external tool servers
interface MCPServer {
  name: string
  url: string
  tools: MCPTool[]
}

// Hermes 已有：`tools/mcp_tool.py` (~1050 lines)
// 支持 MCP server 连接和 tool 调用
```

---

## 7. Feature Flags & A/B Testing

### GrowthBook Pattern

```typescript
// Feature flag gating for gradual rollout
interface FeatureFlag {
  key: string
  enabled: boolean
  rolloutPercentage?: number  // 0-100
  userSegments?: string[]
}

function isFeatureEnabled(flag: string, userId: string): boolean {
  const flag = getFlag(flag)
  if (!flag.enabled) return false
  if (flag.rolloutPercentage) {
    return hash(userId) % 100 < flag.rolloutPercentage
  }
  return true
}
```

**Hermes 已有**: Config-based toolsets enable/disable per platform.

---

## Key Insights for Hermes

| Claude Code Pattern | Hermes Application |
|---------------------|-------------------|
| **Task dependency graph** | Enhance `delegate_task` with `blockedBy` support |
| **Semantic tool search** | Add embedding-based tool discovery to `tools/registry.py` |
| **Three-tier permission** | Already have approval system, could add Tier 1 auto-approve |
| **Plan Mode** | Add `plan_mode` flag to `AIAgent` for analysis-only execution |
| **Compaction triggers** | Already have context_compressor, could add automatic threshold |
| **Stream event typing** | Could improve streaming with explicit event types |

---

## Anti-Patterns to Avoid

| Pattern | Problem | Better Approach |
|---------|---------|-----------------|
| Over-abstraction in tools | 200 lines for simple operation | Single function until complexity needed |
| Silent assumption picking | Tool picks default without asking | Surface assumptions, ask user |
| Drive-by refactoring | "Improving" unrelated code | Surgical changes only |
| Vague success criteria | "Make it work" | Test-first, verifiable goals |
