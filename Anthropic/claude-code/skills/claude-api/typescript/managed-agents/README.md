<!-- BILINGUAL-EN-ZH -->
# Managed Agents - TypeScript / 托管代理——TypeScript

> **Bindings not shown here:** This README covers the most common managed-agents flows for TypeScript. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the TypeScript SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **此处未展示的绑定：**本 README 涵盖 TypeScript 最常用的 managed-agents 流程。如果你需要的类、方法、命名空间、字段或行为此处没有展示，请按 `shared/live-sources.md` 抓取 TypeScript SDK 仓库**或相关文档页面**，而不要凭空猜测。不要从 cURL 请求结构或其他语言的 SDK 外推。

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `agents.create` and pass it to every subsequent `sessions.create`; do not call `agents.create` in the request path. **Recommended:** define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

> **代理是持久的——创建一次，按 ID 引用。**保存 `agents.create` 返回的代理 ID，并在之后每次 `sessions.create` 时传入；不要在请求路径中调用 `agents.create`。**推荐做法：**把代理和环境定义为受版本控制的文件，用 `ant apply` 同步——见 `shared/anthropic-cli.md`（其实时文档 URL 在 `shared/live-sources.md` 中）。CLI 负责控制平面（创建/更新），你的代码负责数据平面（用已存 ID 创建会话）。下面的示例展示了必须在代码中编程式供给时的做法；在生产环境中，create 调用应放在初始化阶段，而不是请求路径中。

## Installation / 安装

```bash
npm install @anthropic-ai/sdk
```

## Client Initialization / 客户端初始化

```typescript
import Anthropic from "@anthropic-ai/sdk";

// Default - resolves credentials from the environment:
// ANTHROPIC_API_KEY, or ANTHROPIC_AUTH_TOKEN, or an `ant auth login` profile.
// Prefer this for local dev; don't hardcode a key.
const client = new Anthropic();

// Explicit API key (only when you must inject a specific key)
const client = new Anthropic({ apiKey: "your-api-key" });
```

---

## Create an Environment / 创建环境

```typescript
const environment = await client.beta.environments.create(
  {
    name: "my-dev-env",
    config: {
      type: "cloud",
      networking: { type: "unrestricted" },
    },
  },
);
console.log(environment.id); // env_...
```

---

## Create an Agent (required first step) / 创建代理（必需的第一步）

> Warning: **There is no inline agent config.** `model`/`system`/`tools` live on the agent object, not the session. Always start with `agents.create()` - the session only takes `agent: { type: "agent", id: agent.id }`.

> 警告：**不存在内联的代理配置。**`model`/`system`/`tools` 位于代理对象上，而不是会话上。始终从 `agents.create()` 开始——会话只接受 `agent: { type: "agent", id: agent.id }`。

### Minimal / 最小示例

```typescript
// 1. Create the agent (reusable, versioned)
const agent = await client.beta.agents.create(
  {
    name: "Coding Assistant",
    model: "claude-opus-5-5",
    tools: [{ type: "agent_toolset_20260401", default_config: { enabled: true } }],
  },
);

// 2. Start a session
const session = await client.beta.sessions.create(
  {
    agent: { type: "agent", id: agent.id, version: agent.version },
    environment_id: environment.id,
  },
);
console.log(session.id, session.status);
console.log(`Trace: https://platform.claude.com/workspaces/default/sessions/${session.id}`); // swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### With system prompt and custom tools / 带系统提示词与自定义工具

```typescript
const agent = await client.beta.agents.create(
  {
    name: "Code Reviewer",
    model: "claude-opus-5-5",
    system: "You are a senior code reviewer.",
    tools: [
      { type: "agent_toolset_20260401", default_config: { enabled: true } },
      {
        type: "custom",
        name: "run_tests",
        description: "Run the test suite",
        input_schema: {
          type: "object",
          properties: {
            test_path: { type: "string", description: "Path to test file" },
          },
          required: ["test_path"],
        },
      },
    ],
  },
);

const session = await client.beta.sessions.create(
  {
    agent: { type: "agent", id: agent.id, version: agent.version },
    environment_id: environment.id,
    title: "Code review session",
    resources: [
      {
        type: "github_repository",
        url: "https://github.com/owner/repo",
        mount_path: "/workspace/repo",
        authorization_token: process.env.GITHUB_TOKEN,
        branch: "main",
      },
    ],
  },
);
```

---

## Send a User Message / 发送用户消息

```typescript
await client.beta.sessions.events.send(
  session.id,
  {
    events: [
      {
        type: "user.message",
        content: [{ type: "text", text: "Review the auth module" }],
      },
    ],
  },
);
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

> 提示：**流式优先：**在发送消息*之前*（或与之并发）打开流。流只送达它打开之后发生的事件——先发消息后开流会导致早期事件以一整批缓冲的形式到达。参见 [转向模式](../../shared/managed-agents-events.md#steering-patterns)。

---

## Define an Outcome (default kickoff for deliverables) / 定义成果（交付物的默认启动方式）

When the session's job is to produce something checkable - an artifact, a report, a PR - kick off with `user.define_outcome` instead of `user.message`: the harness grades each iteration against your rubric and the agent revises until it passes. Send one or the other, never both. See [Outcomes](../../shared/managed-agents-outcomes.md) for the event reference and rubric-writing guidance.

当会话的任务是产出可检验的东西——一个工件、一份报告、一个 PR——时，用 `user.define_outcome` 而不是 `user.message` 来启动：执行框架会按你的评分标准（rubric）对每次迭代打分，代理据此修改直到通过。两者只发其一，绝不同时都发。事件参考与评分标准撰写指南见 [成果（Outcomes）](../../shared/managed-agents-outcomes.md)。

```typescript
const STARTER_RUBRIC = `# Report rubric - starter, tune the criteria
- Output is a single \`report.md\` in /mnt/session/outputs/
- Every claim cites a source URL
- Includes a summary table with one row per competitor
- Prices are current as of the run date and each row says where it was read from
- No placeholder text, TODOs, or empty sections remain
`;

await client.beta.sessions.events.send(
  session.id,
  {
    events: [
      {
        type: "user.define_outcome",
        description: "Write a competitor-pricing report as report.md",
        rubric: { type: "text", content: STARTER_RUBRIC },
        max_iterations: 5, // optional; default 3, max 20
      },
    ],
  },
);
```

---

## Stream Events (SSE) / 流式事件（SSE）

```typescript
// Stream-first: open stream and send concurrently
const [events] = await Promise.all([
  collectStream(session.id),
  client.beta.sessions.events.send(
    session.id,
    { events: [{ type: "user.message", content: [{ type: "text", text: "..." }] }] },
  ),
]);

// Standalone stream iteration:
const stream = await client.beta.sessions.events.stream(
  session.id,
);

for await (const event of stream) {
  switch (event.type) {
    case "agent.message":
      for (const block of event.content) {
        if (block.type === "text") {
          process.stdout.write(block.text);
        }
      }
      break;
    case "agent.custom_tool_use":
      // Custom tool invocation - session is now idle
      console.log(`\nCustom tool call: ${event.name}`);
      console.log(`Input: ${JSON.stringify(event.input)}`);
      break;
    case "session.status_idle":
      console.log("\n--- Agent idle ---");
      break;
    case "session.status_terminated":
      console.log("\n--- Session terminated ---");
      break;
  }
}
```

---

## Provide Custom Tool Result / 提供自定义工具结果

```typescript
await client.beta.sessions.events.send(
  session.id,
  {
    events: [
      {
        type: "user.custom_tool_result",
        custom_tool_use_id: "sevt_abc123",
        content: [{ type: "text", text: "All 42 tests passed." }],
      },
    ],
  },
);
```

---

## Poll Events / 轮询事件

```typescript
const events = await client.beta.sessions.events.list(
  session.id,
);
for (const event of events.data) {
  console.log(`${event.type}: ${event.id}`);
}
```

---

## Full Streaming Loop with Custom Tools / 带自定义工具的完整流式循环

```typescript
function runCustomTool(toolName: string, toolInput: unknown): string {
  if (toolName === "run_tests") {
    // Your tool implementation here
    return "All tests passed.";
  }
  return `Unknown tool: ${toolName}`;
}

async function runSession(client: Anthropic, sessionId: string) {
  while (true) {
    const stream = await client.beta.sessions.events.stream(
      sessionId,
    );

    const toolCalls: Anthropic.Beta.Sessions.BetaManagedAgentsAgentCustomToolUseEvent[] = [];

    for await (const event of stream) {
      if (event.type === "agent.message") {
        for (const block of event.content) {
          if (block.type === "text") {
            process.stdout.write(block.text);
          }
        }
      } else if (event.type === "agent.custom_tool_use") {
        toolCalls.push(event);
      } else if (event.type === "session.status_idle") {
        break;
      } else if (event.type === "session.status_terminated") {
        return;
      }
    }

    if (toolCalls.length === 0) break;

    // Process custom tool calls
    const results = toolCalls.map((call) => ({
      type: "user.custom_tool_result" as const,
      custom_tool_use_id: call.id,
      content: [{ type: "text" as const, text: runCustomTool(call.name, call.input) }],
    }));

    await client.beta.sessions.events.send(
      sessionId,
      { events: results },
    );
  }
}
```

---

## Upload a File / 上传文件

```typescript
import fs from "fs";

const file = await client.beta.files.upload({
  file: fs.createReadStream("data.csv"),
  purpose: "agent",
});

// Use in a session
const session = await client.beta.sessions.create(
  {
    agent: { type: "agent", id: agent.id, version: agent.version },
    environment_id: environment.id,
    resources: [{ type: "file", file_id: file.id, mount_path: "/workspace/data.csv" }],
  },
);
```

---

## List and Download Session Files / 列出并下载会话文件

List files the agent wrote to `/mnt/session/outputs/` during a session, then download them.

列出代理在会话期间写入 `/mnt/session/outputs/` 的文件，然后下载它们。

```typescript
import fs from "fs";

// List files associated with a session
const files = await client.beta.files.list({
  scope_id: session.id,
  betas: ["managed-agents-2026-04-01"],
});
for (const f of files.data) {
  console.log(f.filename, f.size_bytes);

  // Download and save to disk
  const resp = await client.beta.files.download(f.id);
  const buffer = Buffer.from(await resp.arrayBuffer());
  fs.writeFileSync(f.filename, buffer);
}
```

> Tip: There's a brief indexing lag (~1-3s) between `session.status_idle` and output files appearing in `files.list`. Retry once or twice if the list is empty.

> 提示：从 `session.status_idle` 到输出文件出现在 `files.list` 之间有短暂的索引延迟（约 1-3 秒）。如果列表为空，重试一两次。

---

## Session Management / 会话管理

```typescript
// Get session details
const session = await client.beta.sessions.retrieve("sesn_011CZxAbc123Def456");
console.log(session.status, session.usage);

// List sessions
const sessions = await client.beta.sessions.list();

// Delete a session
await client.beta.sessions.delete("sesn_011CZxAbc123Def456");

// Archive a session
await client.beta.sessions.archive("sesn_011CZxAbc123Def456");
```

---

## MCP Server Integration / MCP 服务器集成

```typescript
// Agent declares MCP server (no auth here - auth goes in a vault)
const agent = await client.beta.agents.create({
  name: "MCP Agent",
  model: "claude-opus-5-5",
  mcp_servers: [
    { type: "url", name: "my-tools", url: "https://my-mcp-server.example.com/sse" },
  ],
  tools: [
    { type: "agent_toolset_20260401", default_config: { enabled: true } },
    { type: "mcp_toolset", mcp_server_name: "my-tools" },
  ],
});

// Session attaches vault(s) containing credentials for those MCP server URLs
const session = await client.beta.sessions.create({
  agent: agent.id,
  environment_id: environment.id,
  vault_ids: [vault.id],
});
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

创建保管库（vault）和添加凭证见 `shared/managed-agents-tools.md` 的 §Vaults 一节。
