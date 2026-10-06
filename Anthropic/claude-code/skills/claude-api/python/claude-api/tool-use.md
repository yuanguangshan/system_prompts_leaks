<!-- BILINGUAL-EN-ZH -->
# Tool Use - Python / 工具使用 - Python

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概述（工具定义、工具选择、技巧）参见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Runner (Recommended) / 工具运行器（推荐）

**Beta:** The tool runner is in beta in the Python SDK.

**Beta：**工具运行器在 Python SDK 中处于 beta 阶段。

Use the `@beta_tool` decorator to define tools as typed functions, then pass them to `client.beta.messages.tool_runner()`:

使用 `@beta_tool` 装饰器把工具定义为带类型的函数，然后把它们传给 `client.beta.messages.tool_runner()`：

```python
import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """Get current weather for a location.

    Args:
        location: City and state, e.g., San Francisco, CA.
        unit: Temperature unit, either "celsius" or "fahrenheit".
    """
    # Your implementation here
    return f"72°F and sunny in {location}"

# The tool runner handles the agentic loop automatically
runner = client.beta.messages.tool_runner(
    model="claude-opus-5-5",
    max_tokens=16000,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
)

# Each iteration yields a BetaMessage; iteration stops when Claude is done
for message in runner:
    print(message)
```

For async usage, use `@beta_async_tool` with `async def` functions.

异步用法请将 `@beta_async_tool` 与 `async def` 函数配合使用。

**Key benefits of the tool runner:**

**工具运行器的主要优点：**

- No manual loop - the SDK handles calling tools and feeding results back
  无需手动循环——SDK 负责调用工具并把结果回填
- Type-safe tool inputs via decorators
  通过装饰器获得类型安全的工具输入
- Tool schemas are generated automatically from function signatures
  工具 schema 从函数签名自动生成
- Iteration stops automatically when Claude has no more tool calls
  当 Claude 没有更多工具调用时，迭代自动停止

### Server tools with the tool runner / 配合工具运行器使用服务器工具

The runner's `tools` list accepts raw server-tool definitions (`web_search_20260209`, `web_fetch_20260209`, code execution) alongside decorated tools - pass the literal tool dict; server tools run on Anthropic's servers, so there is no function to implement.

运行器的 `tools` 列表在接受装饰器工具的同时，也接受原始的服务器工具定义（`web_search_20260209`、`web_fetch_20260209`、代码执行）——直接传入字面的工具字典；服务器工具在 Anthropic 的服务器上运行，因此没有需要实现的函数。

**Caution - the runner does not auto-resume `pause_turn` (as of `anthropic` 0.116.0).** A long-running server-tool turn can stop with `stop_reason: "pause_turn"`. The runner only continues after a client tool produces a result, so a paused turn ends the loop and is returned as the final message - no error, no warning, just a silently truncated answer. Unlike the TypeScript runner, the Python runner cannot be resumed mid-loop: it exits unconditionally when no client tool ran, and `runner.append_messages(...)` does not prevent the exit. To handle `pause_turn`, mirror the conversation history as you iterate, then restart the runner with the paused turn appended:

**注意——运行器不会自动恢复 `pause_turn`（截至 `anthropic` 0.116.0）。**长时间运行的服务器工具回合可能以 `stop_reason: "pause_turn"` 停止。运行器只在客户端工具产生结果后才继续，因此被暂停的回合会结束循环并作为最终消息返回——没有错误、没有警告，只有一个被静默截断的答案。与 TypeScript 运行器不同，Python 运行器无法在循环中途恢复：当没有客户端工具运行时它会无条件退出，且 `runner.append_messages(...)` 不能阻止退出。要处理 `pause_turn`，请在迭代时自行镜像会话历史，然后追加被暂停的回合重启运行器：

【评论】此段把一个 beta 功能的已知边界行为（静默截断、不可中途恢复）写进文档，提醒调用方自行兜底；使用 beta API 时这类注意事项值得逐条核对版本号。

```python
messages = [{"role": "user", "content": user_input}]

max_restarts = 5  # cap pause_turn restarts, mirroring max_continuations advice
restarts = 0
while True:
    runner = client.beta.messages.tool_runner(
        model="claude-opus-5-5",
        max_tokens=16000,
        tools=tools,  # may mix @beta_tool functions and server-tool definitions
        messages=messages,
    )
    last = None
    for message in runner:
        last = message
        # Mirror the history - the runner keeps its own copy and does not expose it
        messages.append({"role": "assistant", "content": message.content})
        tool_response = runner.generate_tool_call_response()  # cached; tools still run once
        if tool_response is not None:
            messages.append(tool_response)
    if last is None or last.stop_reason != "pause_turn":
        break
    restarts += 1
    if restarts > max_restarts:
        raise RuntimeError("giving up: turn still paused after max_restarts")
    # Paused mid-turn: `messages` already ends with the paused assistant
    # turn, so the next runner resumes it
```

Alternatively, use the manual loop below, which handles `pause_turn` explicitly.

或者，使用下面的手动循环，它显式处理 `pause_turn`。

---

## MCP Tool Conversion Helpers / MCP 工具转换辅助函数

**Beta.** Convert [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) tools, prompts, and resources to Anthropic API types for use with the tool runner. Requires `pip install anthropic[mcp]` (Python 3.10+).

**Beta。**把 [MCP（Model Context Protocol）](https://modelcontextprotocol.io/) 的工具、提示与资源转换为 Anthropic API 类型，以配合工具运行器使用。需要 `pip install anthropic[mcp]`（Python 3.10+）。

> **Note:** The Claude API also supports an `mcp_servers` parameter that lets Claude connect directly to remote MCP servers. Use these helpers instead when you need local MCP servers, prompts, resources, or more control over the MCP connection.

> **注意：**Claude API 还支持 `mcp_servers` 参数，让 Claude 直接连接远程 MCP 服务器。当你需要本地 MCP 服务器、提示、资源，或需要对 MCP 连接的更多控制时，改用这些辅助函数。

### MCP Tools with Tool Runner / 在工具运行器中使用 MCP 工具

```python
from anthropic import AsyncAnthropic
from anthropic.lib.tools.mcp import async_mcp_tool
from mcp import ClientSession
from mcp.client.stdio import stdio_client, StdioServerParameters

client = AsyncAnthropic()

async with stdio_client(StdioServerParameters(command="mcp-server")) as (read, write):
    async with ClientSession(read, write) as mcp_client:
        await mcp_client.initialize()

        tools_result = await mcp_client.list_tools()
        # tool_runner is sync - returns the runner, not a coroutine
        runner = client.beta.messages.tool_runner(
            model="claude-opus-5-5",
            max_tokens=16000,
            messages=[{"role": "user", "content": "Use the available tools"}],
            tools=[async_mcp_tool(t, mcp_client) for t in tools_result.tools],
        )
        async for message in runner:
            print(message)
```

For sync usage, use `mcp_tool` instead of `async_mcp_tool`.

同步用法请用 `mcp_tool` 而非 `async_mcp_tool`。

### MCP Prompts / MCP 提示（Prompts）

```python
from anthropic.lib.tools.mcp import mcp_message

prompt = await mcp_client.get_prompt(name="my-prompt")
response = await client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[mcp_message(m) for m in prompt.messages],
)
```

### MCP Resources as Content / MCP 资源作为内容

```python
from anthropic.lib.tools.mcp import mcp_resource_to_content

resource = await mcp_client.read_resource(uri="file:///path/to/doc.txt")
response = await client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            mcp_resource_to_content(resource),
            {"type": "text", "text": "Summarize this document"},
        ],
    }],
)
```

### Upload MCP Resources as Files / 将 MCP 资源上传为文件

```python
from anthropic.lib.tools.mcp import mcp_resource_to_file

resource = await mcp_client.read_resource(uri="file:///path/to/data.json")
uploaded = await client.beta.files.upload(file=mcp_resource_to_file(resource))
```

Conversion functions raise `UnsupportedMCPValueError` if an MCP value cannot be converted (e.g., unsupported content types like audio, unsupported MIME types).

当 MCP 值无法转换时（如音频等不支持的内容类型、不支持的 MIME 类型），转换函数会抛出 `UnsupportedMCPValueError`。

---

## Manual Agentic Loop / 手动智能体循环

Prefer the tool runner above. Drop to a manual loop only when you need control the runner does not expose (e.g., a custom transport, request shapes the SDK cannot build, or avoiding a beta dependency - the runner is beta). Human-in-the-loop approval does *not* require a manual loop - gate inside the tool function (return a "user declined" result) or inspect pending `tool_use` blocks in the `for message in runner:` body and call `runner.set_messages_params()`.

优先使用上面的工具运行器。只有当需要运行器未暴露的控制能力时（如自定义传输、SDK 无法构建的请求形态，或避免 beta 依赖——运行器是 beta 功能）才退回手动循环。人在环中的审批*并不*需要手动循环——在工具函数内部加门（返回"用户拒绝"的结果），或在 `for message in runner:` 循环体中检查待处理的 `tool_use` 块并调用 `runner.set_messages_params()`。

If you do need a manual loop:

如果你确实需要手动循环：

```python
import anthropic

client = anthropic.Anthropic()
tools = [...]  # Your tool definitions
messages = [{"role": "user", "content": user_input}]

# Agentic loop: keep going until Claude stops calling tools
while True:
    response = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=16000,
        tools=tools,
        messages=messages
    )

    # If Claude is done (no more tool calls), break
    if response.stop_reason == "end_turn":
        break

    # Server-side tool hit iteration limit; re-send to continue
    if response.stop_reason == "pause_turn":
        messages = [
            {"role": "user", "content": user_input},
            {"role": "assistant", "content": response.content},
        ]
        continue

    # Extract tool use blocks from the response
    tool_use_blocks = [b for b in response.content if b.type == "tool_use"]

    # Append assistant's response (including tool_use blocks)
    messages.append({"role": "assistant", "content": response.content})

    # Execute each tool and collect results
    tool_results = []
    for tool in tool_use_blocks:
        result = execute_tool(tool.name, tool.input)  # Your implementation
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": tool.id,  # Must match the tool_use block's id
            "content": result
        })

    # Append tool results as a user message
    messages.append({"role": "user", "content": tool_results})

# Final response text
final_text = next(b.text for b in response.content if b.type == "text")
```

---

## Handling Tool Results / 处理工具结果

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)

for block in response.content:
    if block.type == "tool_use":
        tool_name = block.name
        tool_input = block.input
        tool_use_id = block.id

        result = execute_tool(tool_name, tool_input)

        followup = client.messages.create(
            model="claude-opus-5-5",
            max_tokens=16000,
            tools=tools,
            messages=[
                {"role": "user", "content": "What's the weather in Paris?"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": tool_use_id,
                        "content": result
                    }]
                }
            ]
        )
```

---

## Multiple Tool Calls / 多个工具调用

```python
tool_results = []

for block in response.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result
        })

# Send all results back at once
if tool_results:
    followup = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=16000,
        tools=tools,
        messages=[
            *previous_messages,
            {"role": "assistant", "content": response.content},
            {"role": "user", "content": tool_results}
        ]
    )
```

---

## Error Handling in Tool Results / 工具结果中的错误处理

```python
tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "Error: Location 'xyz' not found. Please provide a valid city name.",
    "is_error": True
}
```

---

## Tool Choice / 工具选择

`tool_choice` is `{"type": "auto"}` by default. Forcing a call (`{"type": "any"}` or `{"type": "tool", "name": ...}`) returns a 400 on Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and Claude Mythos 5.1; Claude Opus 5, Claude Sonnet 5, and older models accept it. Steer with the prompt instead, and keep the schema guarantee with `strict: true`:

`tool_choice` 默认为 `{"type": "auto"}`。强制调用（`{"type": "any"}` 或 `{"type": "tool", "name": ...}`）在 Claude Opus 5.5、Claude Sonnet 5.5、Claude Fable 5.1 和 Claude Mythos 5.1 上会返回 400；Claude Opus 5、Claude Sonnet 5 及更早的模型则接受。应改为用提示词引导，并用 `strict: true` 保留 schema 保证：

【评论】新版模型不再支持强制 `tool_choice`，转而推荐"提示词引导 + strict schema"，体现了在模型自主性与调用确定性之间的权衡变化；跨模型兼容的代码需注意这一行为差异。

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    tools=[{**tool, "strict": True} for tool in tools],  # schemas must set additionalProperties: false
    messages=[{"role": "user", "content": "What's the weather in Paris? Use the get_weather tool."}]
)
# auto does not guarantee a call - check for a tool_use block and re-prompt if none came back
```

---

## Code Execution / 代码执行

### Basic Usage / 基本用法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": "Calculate the mean and standard deviation of [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]"
    }],
    tools=[{
        "type": "code_execution_20260120",
        "name": "code_execution"
    }]
)

for block in response.content:
    if block.type == "text":
        print(block.text)
    elif block.type == "bash_code_execution_tool_result":
        print(f"stdout: {block.content.stdout}")
```

### Upload Files for Analysis / 上传文件供分析

```python
# 1. Upload a file
uploaded = client.beta.files.upload(file=open("sales_data.csv", "rb"))

# 2. Pass to code execution via container_upload block
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Analyze this sales data. Show trends and create a visualization."},
            {"type": "container_upload", "file_id": uploaded.id}
        ]
    }],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### Retrieve Generated Files / 取回生成的文件

```python
import os

OUTPUT_DIR = "./claude_outputs"
os.makedirs(OUTPUT_DIR, exist_ok=True)

for block in response.content:
    if block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result" and result.content:
            for file_ref in result.content:
                if file_ref.type == "bash_code_execution_output":
                    metadata = client.beta.files.retrieve_metadata(file_ref.file_id)
                    file_content = client.beta.files.download(file_ref.file_id)
                    # Use basename to prevent path traversal; validate result
                    safe_name = os.path.basename(metadata.filename)
                    if not safe_name or safe_name in (".", ".."):
                        print(f"Skipping invalid filename: {metadata.filename}")
                        continue
                    output_path = os.path.join(OUTPUT_DIR, safe_name)
                    file_content.write_to_file(output_path)
                    print(f"Saved: {output_path}")
```

### Container Reuse / 容器复用

```python
# First request: set up environment
response1 = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{"role": "user", "content": "Install tabulate and create data.json with sample data"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)

# Get container ID from response
container_id = response1.container.id

# Second request: reuse the same container
response2 = client.messages.create(
    container=container_id,
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{"role": "user", "content": "Read data.json and display as a formatted table"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

### Response Structure / 响应结构

```python
for block in response.content:
    if block.type == "text":
        print(block.text)  # Claude's explanation
    elif block.type == "server_tool_use":
        print(f"Running: {block.name} - {block.input}")  # What Claude is doing
    elif block.type == "bash_code_execution_tool_result":
        result = block.content
        if result.type == "bash_code_execution_result":
            if result.return_code == 0:
                print(f"Output: {result.stdout}")
            else:
                print(f"Error: {result.stderr}")
        else:
            print(f"Tool error: {result.error_code}")
    elif block.type == "text_editor_code_execution_tool_result":
        print(f"File operation: {block.content}")
```

---

## Memory Tool / 记忆工具

### Basic Usage / 基本用法

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{"role": "user", "content": "Remember that my preferred language is Python."}],
    tools=[{"type": "memory_20250818", "name": "memory"}],
)
```

### SDK Memory Helper / SDK 记忆辅助类

Subclass `BetaAbstractMemoryTool`:

继承 `BetaAbstractMemoryTool`：

```python
from anthropic.lib.tools import BetaAbstractMemoryTool

class MyMemoryTool(BetaAbstractMemoryTool):
    def view(self, command): ...
    def create(self, command): ...
    def str_replace(self, command): ...
    def insert(self, command): ...
    def delete(self, command): ...
    def rename(self, command): ...

memory = MyMemoryTool()

# Use with tool runner
runner = client.beta.messages.tool_runner(
    model="claude-opus-5-5",
    max_tokens=16000,
    tools=[memory],
    messages=[{"role": "user", "content": "Remember my preferences"}],
)

for message in runner:
    print(message)
```

For full implementation examples, use WebFetch:

完整实现示例请用 WebFetch 获取：

- `https://github.com/anthropics/anthropic-sdk-python/blob/main/examples/memory/basic.py`

---

## Structured Outputs / 结构化输出

### JSON Outputs (Pydantic - Recommended) / JSON 输出（Pydantic——推荐）

```python
from pydantic import BaseModel
from typing import List
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool

client = anthropic.Anthropic()

response = client.messages.parse(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": "Extract: Jane Doe (jane@co.com) wants Enterprise, interested in API and SDKs, wants a demo."
    }],
    output_format=ContactInfo,
)

# response.parsed_output is a validated ContactInfo instance
contact = response.parsed_output
print(contact.name)           # "Jane Doe"
print(contact.interests)      # ["API", "SDKs"]
```

### Raw Schema / 原始 Schema

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": "Extract info: John Smith (john@example.com) wants the Enterprise plan."
    }],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "email": {"type": "string"},
                    "plan": {"type": "string"},
                    "demo_requested": {"type": "boolean"}
                },
                "required": ["name", "email", "plan", "demo_requested"],
                "additionalProperties": False
            }
        }
    }
)

import json
# output_config.format guarantees the first block is text with valid JSON
text = next(b.text for b in response.content if b.type == "text")
data = json.loads(text)
```

### Strict Tool Use / 严格工具使用

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{"role": "user", "content": "Book a flight to Tokyo for 2 passengers on March 15"}],
    tools=[{
        "name": "book_flight",
        "description": "Book a flight to a destination",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"},
                "passengers": {"type": "integer", "enum": [1, 2, 3, 4, 5, 6, 7, 8]}
            },
            "required": ["destination", "date", "passengers"],
            "additionalProperties": False
        }
    }]
)
```

### Using Both Together / 两者结合使用

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{"role": "user", "content": "Plan a trip to Paris next month"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "summary": {"type": "string"},
                    "next_steps": {"type": "array", "items": {"type": "string"}}
                },
                "required": ["summary", "next_steps"],
                "additionalProperties": False
            }
        }
    },
    tools=[{
        "name": "search_flights",
        "description": "Search for available flights",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "destination": {"type": "string"},
                "date": {"type": "string", "format": "date"}
            },
            "required": ["destination", "date"],
            "additionalProperties": False
        }
    }]
)
```
