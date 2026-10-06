<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Python / 托管智能体 - Python

> **Bindings not shown here:** This README covers the most common managed-agents flows for Python. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the Python SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **此处未展示的绑定：** 本 README 涵盖 Python 上最常见的托管智能体（managed-agents）流程。如果你需要的类、方法、命名空间、字段或行为未在此展示，请通过 WebFetch 访问 `shared/live-sources.md` 中的 Python SDK 仓库**或相关文档页面**，而不是凭猜测行事。不要从 cURL 形态或其他语言的 SDK 进行外推。

【评论】该段落是典型的防幻觉指令：要求模型在缺少依据时实时抓取官方源码或文档，而不是基于模式相似性编造 API 形态。

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `agents.create` and pass it to every subsequent `sessions.create`; do not call `agents.create` in the request path. **Recommended:** define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

> **智能体是持久的 —— 创建一次，按 ID 引用。** 保存 `agents.create` 返回的智能体 ID，并在之后每次 `sessions.create` 时传入它；不要在请求路径中调用 `agents.create`。**推荐做法：** 把智能体和环境定义为受版本控制的文件，用 `ant apply` 同步 —— 见 `shared/anthropic-cli.md`（其实时文档 URL 在 `shared/live-sources.md` 中）。CLI 负责控制平面（创建/更新）；你的代码负责数据平面（用已保存的 ID 发起会话）。下面的示例展示了必须在代码中以编程方式创建资源时的写法；在生产环境中，创建调用应属于初始化设置，而不是请求路径。

## Installation / 安装

```bash
pip install anthropic
```

## Client Initialization / 客户端初始化

```python
import anthropic

# Default - resolves credentials from the environment:
# ANTHROPIC_API_KEY, or ANTHROPIC_AUTH_TOKEN, or an `ant auth login` profile.
# Prefer this for local dev; don't hardcode a key.
client = anthropic.Anthropic()

# Explicit API key (only when you must inject a specific key)
client = anthropic.Anthropic(api_key="your-api-key")
```

---

## Create an Environment / 创建环境

```python
environment = client.beta.environments.create(
    name="my-dev-env",
    config={
        "type": "cloud",
        "networking": {"type": "unrestricted"},
    },
)
print(environment.id)  # env_...
```

---

## Create an Agent (required first step) / 创建智能体（必需的第一步）

> Warning: **There is no inline agent config.** `model`/`system`/`tools` live on the agent object, not the session. Always start with `agents.create()` - the session only takes `agent={"type": "agent", "id": agent.id}`.

> 警告：**不存在内联的智能体配置。** `model`/`system`/`tools` 位于智能体对象上，而不是会话上。始终从 `agents.create()` 开始 —— 会话只接受 `agent={"type": "agent", "id": agent.id}`。

### Minimal / 最小示例

```python
# 1. Create the agent (reusable, versioned)
agent = client.beta.agents.create(
    name="Coding Assistant",
    model="claude-opus-5-5",
    tools=[{"type": "agent_toolset_20260401", "default_config": {"enabled": True}}],
)

# 2. Start a session
session = client.beta.sessions.create(
    agent={"type": "agent", "id": agent.id, "version": agent.version},
    environment_id=environment.id,
)
print(session.id, session.status)
print(f"Trace: https://platform.claude.com/workspaces/default/sessions/{session.id}")  # swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### With system prompt and custom tools / 带系统提示词与自定义工具

```python
import os

agent = client.beta.agents.create(
    name="Code Reviewer",
    model="claude-opus-5-5",
    system="You are a senior code reviewer.",
    tools=[
        {"type": "agent_toolset_20260401"},
        {
            "type": "custom",
            "name": "run_tests",
            "description": "Run the test suite",
            "input_schema": {
                "type": "object",
                "properties": {
                    "test_path": {"type": "string", "description": "Path to test file"}
                },
                "required": ["test_path"],
            },
        },
    ],
)

session = client.beta.sessions.create(
    agent={"type": "agent", "id": agent.id, "version": agent.version},
    environment_id=environment.id,
    title="Code review session",
    resources=[
        {
            "type": "github_repository",
            "url": "https://github.com/owner/repo",
            "mount_path": "/workspace/repo",
            "authorization_token": os.environ["GITHUB_TOKEN"],
            "branch": "main",
        }
    ],
)
```

---

## Send a User Message / 发送用户消息

```python
client.beta.sessions.events.send(
    session_id=session.id,
    events=[
        {
            "type": "user.message",
            "content": [{"type": "text", "text": "Review the auth module"}],
        }
    ],
)
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

> 提示：**流式优先：** 在发送消息*之前*（或同时）打开流。流只会传递在它打开之后发生的事件 —— 先发送后开流意味着早期事件会以一批缓冲的形式一次性到达。见 [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns)。

---

## Define an Outcome (default kickoff for deliverables) / 定义成果（交付物的默认启动方式）

When the session's job is to produce something checkable - an artifact, a report, a PR - kick off with `user.define_outcome` instead of `user.message`: the harness grades each iteration against your rubric and the agent revises until it passes. Send one or the other, never both. See [Outcomes](../../shared/managed-agents-outcomes.md) for the event reference and rubric-writing guidance.

当会话的任务是产出可检验的东西 —— 一个产物、一份报告、一个 PR —— 时，用 `user.define_outcome` 而不是 `user.message` 来启动：执行框架（harness）会按你的评分标准（rubric）对每次迭代打分，智能体不断修订直到通过。二者只发其一，绝不同时发送。事件参考和评分标准撰写指南见 [Outcomes](../../shared/managed-agents-outcomes.md)。

```python
STARTER_RUBRIC = """# Report rubric - starter, tune the criteria
- Output is a single `report.md` in /mnt/session/outputs/
- Every claim cites a source URL
- Includes a summary table with one row per competitor
- Prices are current as of the run date and each row says where it was read from
- No placeholder text, TODOs, or empty sections remain
"""

client.beta.sessions.events.send(
    session_id=session.id,
    events=[
        {
            "type": "user.define_outcome",
            "description": "Write a competitor-pricing report as report.md",
            "rubric": {"type": "text", "content": STARTER_RUBRIC},
            "max_iterations": 5,  # optional; default 3, max 20
        }
    ],
)
```

---

## Stream Events (SSE) / 流式接收事件（SSE）

```python
import json

# Stream-first: open stream, then send while stream is live
with client.beta.sessions.events.stream(
    session_id=session.id,
) as stream:
    client.beta.sessions.events.send(
        session_id=session.id,
        events=[{"type": "user.message", "content": [{"type": "text", "text": "..."}]}],
    )
    for event in stream:
        ...  # process events

# Standalone stream iteration:
with client.beta.sessions.events.stream(
    session_id=session.id,
) as stream:
    for event in stream:
        if event.type == "agent.message":
            for block in event.content:
                if block.type == "text":
                    print(block.text, end="", flush=True)
        elif event.type == "agent.custom_tool_use":
            # Custom tool invocation - session is now idle
            print(f"\nCustom tool call: {event.name}")
            print(f"Input: {json.dumps(event.input)}")
            # Send result back (see below)
        elif event.type == "session.status_idle":
            print("\n--- Agent idle ---")
        elif event.type == "session.status_terminated":
            print("\n--- Session terminated ---")
            break
```

---

## Provide Custom Tool Result / 提供自定义工具结果

```python
client.beta.sessions.events.send(
    session_id=session.id,
    events=[
        {
            "type": "user.custom_tool_result",
            "custom_tool_use_id": "sevt_abc123",
            "content": [{"type": "text", "text": "All 42 tests passed."}],
        }
    ],
)
```

---

## Poll Events / 轮询事件

```python
events = client.beta.sessions.events.list(
    session_id=session.id,
)
for event in events.data:
    print(f"{event.type}: {event.id}")
```

> Warning: **Prefer the SDK over raw `requests`/`httpx`.** If you hand-roll a poll loop, don't assume `timeout=(5, 60)` or `httpx.Timeout(120)` caps total call duration - both are **per-chunk** read timeouts (reset on every byte), so a trickling response can block forever. For a hard wall-clock deadline, track `time.monotonic()` at the loop level and bail explicitly, or wrap with `asyncio.wait_for()`. See [Receiving Events](../../shared/managed-agents-events.md#receiving-events).

> 警告：**优先使用 SDK 而非原生 `requests`/`httpx`。** 如果你自己手写轮询循环，不要以为 `timeout=(5, 60)` 或 `httpx.Timeout(120)` 限制了整个调用的总时长 —— 两者都是**按数据块（per-chunk）** 计的读取超时（每收到一个字节就重置），因此一个缓慢滴流的响应可能永远阻塞。若需要硬性的挂钟时间期限，请在循环层面跟踪 `time.monotonic()` 并显式退出，或用 `asyncio.wait_for()` 包裹。见 [Receiving Events](../../shared/managed-agents-events.md#receiving-events)。

---

## Full Streaming Loop with Custom Tools / 带自定义工具的完整流式循环

```python
import json


def run_custom_tool(tool_name: str, tool_input: dict) -> str:
    """Execute a custom tool and return the result."""
    if tool_name == "run_tests":
        # Your tool implementation here
        return "All tests passed."
    return f"Unknown tool: {tool_name}"


def run_session(client, session_id: str):
    """Stream events and handle custom tool calls."""
    while True:
        with client.beta.sessions.events.stream(
            session_id=session_id,
        ) as stream:
            tool_calls = []
            for event in stream:
                if event.type == "agent.message":
                    for block in event.content:
                        if block.type == "text":
                            print(block.text, end="", flush=True)
                elif event.type == "agent.custom_tool_use":
                    tool_calls.append(event)
                elif event.type == "session.status_idle":
                    break
                elif event.type == "session.status_terminated":
                    return

        if not tool_calls:
            break

        # Process custom tool calls
        results = []
        for call in tool_calls:
            result = run_custom_tool(call.name, call.input)
            results.append({
                "type": "user.custom_tool_result",
                "custom_tool_use_id": call.id,
                "content": [{"type": "text", "text": result}],
            })

        client.beta.sessions.events.send(
            session_id=session_id,
            events=results,
        )
```

---

## Upload a File / 上传文件

```python
with open("data.csv", "rb") as f:
    file = client.beta.files.upload(
        file=f,
    )

# Use in a session
session = client.beta.sessions.create(
    agent={"type": "agent", "id": agent.id, "version": agent.version},
    environment_id=environment.id,
    resources=[{"type": "file", "file_id": file.id, "mount_path": "/workspace/data.csv"}],
)
```

---

## List and Download Session Files / 列出并下载会话文件

List files the agent wrote to `/mnt/session/outputs/` during a session, then download them.

列出智能体在会话期间写入 `/mnt/session/outputs/` 的文件，然后下载它们。

```python
# List files associated with a session
files = client.beta.files.list(
    scope_id=session.id,
    betas=["managed-agents-2026-04-01"],
)
for f in files.data:
    print(f.filename, f.size_bytes)
    # Download each file and save to disk
    file_content = client.beta.files.download(f.id)
    file_content.write_to_file(f.filename)
```

> Tip: There's a brief indexing lag (~1-3s) between `session.status_idle` and output files appearing in `files.list`. Retry once or twice if the list is empty.

> 提示：从 `session.status_idle` 到输出文件出现在 `files.list` 之间有短暂的索引延迟（约 1-3 秒）。如果列表为空，可重试一到两次。

---

## Session Management / 会话管理

```python
# Get session details
session = client.beta.sessions.retrieve(session_id="sesn_011CZxAbc123Def456")
print(session.status, session.usage)

# List sessions
sessions = client.beta.sessions.list()

# Delete a session
client.beta.sessions.delete(session_id="sesn_011CZxAbc123Def456")

# Archive a session
client.beta.sessions.archive(session_id="sesn_011CZxAbc123Def456")
```

---

## MCP Server Integration / MCP 服务器集成

```python
# Agent declares MCP server (no auth here - auth goes in a vault)
agent = client.beta.agents.create(
    name="MCP Agent",
    model="claude-opus-5-5",
    mcp_servers=[
        {"type": "url", "name": "my-tools", "url": "https://my-mcp-server.example.com/sse"},
    ],
    tools=[
        {"type": "agent_toolset_20260401", "default_config": {"enabled": True}},
        {"type": "mcp_toolset", "mcp_server_name": "my-tools"},
    ],
)

# Session attaches vault(s) containing credentials for those MCP server URLs
session = client.beta.sessions.create(
    agent=agent.id,
    environment_id=environment.id,
    vault_ids=[vault.id],
)
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

创建保管库和添加凭据见 `shared/managed-agents-tools.md` §Vaults。
