<!-- BILINGUAL-EN-ZH -->

# Managed Agents - Tools & Skills / Managed Agents - 工具与技能

## Tools / 工具

### Server tools vs client tools / 服务端工具与客户端工具

| Type | Who runs it | How it works |
|---|---|---|
| **Prebuilt Claude Agent tools** (`agent_toolset_20260401`) | Anthropic, on the session's container (for `cloud` envs; for `self_hosted`, **your** worker supplies and runs the file/bash tools - see `shared/managed-agents-self-hosted-sandboxes.md`). `web_search` / `web_fetch` always run on Anthropic's servers, in both environment types. | File ops, bash, web search, etc. Enable all at once or configure individually with `enabled: true/false`; restrict the web tools with `allowed_domains` / `blocked_domains`. |
| **MCP tools** (`mcp_toolset`) | Anthropic's orchestration layer | Capabilities exposed by connected MCP servers. Grant access per-server via the toolset. |
| **Custom tools** | **You** - your application handles the call and returns results | Agent emits a `agent.custom_tool_use` event, session goes `idle`, you send back a `user.custom_tool_result` event. |

| 类型 | 谁来运行 | 工作方式 |
|---|---|---|
| **预构建 Claude Agent 工具**（`agent_toolset_20260401`） | Anthropic，在会话的容器上（对 `cloud` 环境；对 `self_hosted`，由**你的** worker 提供并运行文件/bash 工具——见 `shared/managed-agents-self-hosted-sandboxes.md`）。`web_search` / `web_fetch` 在两种环境类型下都始终运行在 Anthropic 的服务器上。 | 文件操作、bash、网络搜索等。可一次全部启用，或用 `enabled: true/false` 逐个配置；用 `allowed_domains` / `blocked_domains` 限制网络工具。 |
| **MCP 工具**（`mcp_toolset`） | Anthropic 的编排层 | 已连接 MCP 服务器暴露的能力。通过工具集按服务器授予访问。 |
| **自定义工具** | **你**——你的应用处理调用并返回结果 | 代理发出 `agent.custom_tool_use` 事件，会话转为 `idle`，你回发一个 `user.custom_tool_result` 事件。 |

**Recommendation:** Enable all prebuilt tools via `agent_toolset_20260401`, then disable individually as needed.

**建议：** 通过 `agent_toolset_20260401` 启用全部预构建工具，之后按需逐个禁用。

**Versioning:** The toolset is a versioned, static resource. When underlying tools change, a new toolset version is created (hence `_20260401`) so you always know exactly what you're getting.

**版本化：** 工具集是带版本的静态资源。当底层工具变化时会创建新的工具集版本（因此有 `_20260401`），让你始终确切知道自己得到的是什么。

### Agent Toolset / Agent 工具集

The `agent_toolset_20260401` provides these built-in tools:

`agent_toolset_20260401` 提供这些内置工具：

| Tool                   | Description                              |
| ---------------------- | ---------------------------------------- |
| `bash` | Execute bash commands in a shell session |
| `read` | Read a file from the local filesystem, including text, images, PDFs, and Jupyter notebooks |
| `write` | Write a file to the local filesystem |
| `edit` | Perform string replacement in a file |
| `glob` | Fast file pattern matching using glob patterns |
| `grep` | Text search using regex patterns |
| `web_fetch` | Fetch content from a URL |
| `web_search` | Search the web for information |

| 工具 | 说明 |
| ---------------------- | ---------------------------------------- |
| `bash` | 在 shell 会话中执行 bash 命令 |
| `read` | 从本地文件系统读取文件，包括文本、图片、PDF 和 Jupyter notebook |
| `write` | 向本地文件系统写入文件 |
| `edit` | 在文件中执行字符串替换 |
| `glob` | 使用 glob 模式进行快速文件匹配 |
| `grep` | 使用正则表达式进行文本搜索 |
| `web_fetch` | 从 URL 获取内容 |
| `web_search` | 在网络上搜索信息 |

Enable the full toolset:

启用完整工具集：

```json
{
  "tools": [
    { "type": "agent_toolset_20260401" }
  ]
}
```

### Per-Tool Configuration / 按工具配置

Override defaults for individual tools. This example enables everything except bash:

为单个工具覆盖默认值。这个例子启用除 bash 之外的一切：

```json
{
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "enabled": true },
      "configs": [
        { "name": "bash", "enabled": false }
      ]
    }
  ]
}
```

| Field | Required | Description |
|---|---|---|
| `type` | Yes | `"agent_toolset_20260401"` |
| `default_config` | No | Applied to all tools. `{ "enabled": bool, "permission_policy": {...} }` |
| `configs` | No | Per-tool overrides: `[{ "name": "...", "type": "...", "enabled": bool, "permission_policy": {...} }]`. `name` identifies the tool (values from the table above); `type` is optional in requests (same value as `name`; the server infers it) and always present in responses. `web_search` / `web_fetch` entries also accept web settings - see § Web search & web fetch settings below. |

| 字段 | 必需 | 说明 |
|---|---|---|
| `type` | 是 | `"agent_toolset_20260401"` |
| `default_config` | 否 | 应用于所有工具。`{ "enabled": bool, "permission_policy": {...} }` |
| `configs` | 否 | 按工具覆盖：`[{ "name": "...", "type": "...", "enabled": bool, "permission_policy": {...} }]`。`name` 标识工具（取上表中的值）；`type` 在请求中可选（与 `name` 相同；服务器会推断），在响应中始终存在。`web_search` / `web_fetch` 条目还接受 web 设置——见下文 § Web search & web fetch settings。 |

> **Typed SDKs:** each `configs` entry is a member of a union with one member per built-in tool (eight: `BetaManagedAgentsWebFetchToolConfigParams`, `...WebSearchToolConfigParams`, `...BashToolConfigParams`, ...), discriminated by `type`. Python/TypeScript/Ruby dicts and hashes with just `name` + `enabled` + `permission_policy` are unchanged. In Go, Java, C#, and PHP, `configs` is the union itself - build each entry from its per-tool type (Go: `BetaManagedAgentsAgentToolConfigUnionParamsUnion{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{...}}` - the arms are `OfBash` / `OfRead` / `OfWrite` / `OfEdit` / `OfGlob` / `OfGrep` / `OfWebFetch` / `OfWebSearch`; Java: `.addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()...build())`; C#: `new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false }`; PHP: `BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false)`). Code written against an SDK where all tools shared one config type must update how it constructs entries.
> **类型化 SDK：** 每个 `configs` 条目是一个联合类型中的一员，每个内置工具一员（共八个：`BetaManagedAgentsWebFetchToolConfigParams`、`...WebSearchToolConfigParams`、`...BashToolConfigParams` 等），以 `type` 判别。Python/TypeScript/Ruby 中只含 `name` + `enabled` + `permission_policy` 的字典和哈希不变。在 Go、Java、C# 和 PHP 中，`configs` 就是联合类型本身——从每个工具各自的类型构建条目（Go：`BetaManagedAgentsAgentToolConfigUnionParamsUnion{OfWebFetch: &anthropic.BetaManagedAgentsWebFetchToolConfigParams{...}}`——分支为 `OfBash` / `OfRead` / `OfWrite` / `OfEdit` / `OfGlob` / `OfGrep` / `OfWebFetch` / `OfWebSearch`；Java：`.addConfig(BetaManagedAgentsWebFetchToolConfigParams.builder()...build())`；C#：`new BetaManagedAgentsWebFetchToolConfigParams { Enabled = false }`；PHP：`BetaManagedAgentsWebFetchToolConfigParams::with(enabled: false)`）。针对"所有工具共用一个配置类型"的旧 SDK 编写的代码必须更新其构建条目的方式。

### Permission Policies / 权限策略

Control whether server-executed tools (agent toolset + MCP) run automatically, wait for your approval, or have each call evaluated by the server. Does not apply to custom tools (your application executes those).

控制由服务端执行的工具（agent 工具集 + MCP）是自动运行、等待你的批准，还是由服务器评估每次调用。不适用于自定义工具（那些由你的应用执行）。

| Policy | Behavior |
|---|---|
| `always_allow` | Tool executes automatically. Default for the agent toolset. |
| `always_ask` | Session emits `session.status_idle` (`stop_reason.type: requires_action`) and pauses until you send a `user.tool_confirmation` event. Default for MCP toolsets. |
| `auto` | The server evaluates each call (tool + input + session content so far) and **runs it, denies it, or pauses for your approval**. Neither toolset kind defaults to `auto`. See § `auto` below. |

| 策略 | 行为 |
|---|---|
| `always_allow` | 工具自动执行。agent 工具集的默认值。 |
| `always_ask` | 会话发出 `session.status_idle`（`stop_reason.type: requires_action`）并暂停，直到你发送 `user.tool_confirmation` 事件。MCP 工具集的默认值。 |
| `auto` | 服务器评估每次调用（工具 + 输入 + 截至目前的会话内容）并**运行它、拒绝它或暂停等待你的批准**。两种工具集都不默认为 `auto`。见下文 § `auto`。 |

```json
{
  "type": "agent_toolset_20260401",
  "default_config": {
    "enabled": true,
    "permission_policy": { "type": "always_allow" }
  },
  "configs": [
    { "name": "bash", "permission_policy": { "type": "always_ask" } }
  ]
}
```

**Responding to `always_ask`** (and to `auto` calls that pause): send a `user.tool_confirmation` event with `tool_use_id` set to the **event ID** (`sevt_...`, not a `toolu_` ID) of the triggering `agent.tool_use` / `agent.mcp_tool_use` event. Several confirmations can go in one `events` request:

**响应 `always_ask`**（以及暂停的 `auto` 调用）：发送 `user.tool_confirmation` 事件，把 `tool_use_id` 设为触发它的 `agent.tool_use` / `agent.mcp_tool_use` 事件的**事件 ID**（`sevt_...`，不是 `toolu_` ID）。多个确认可以放进一个 `events` 请求：

```js
{ "type": "user.tool_confirmation", "tool_use_id": "sevt_abc123", "result": "allow" }
{ "type": "user.tool_confirmation", "tool_use_id": "sevt_def456", "result": "deny", "deny_message": "Read .env.example instead" }
```

The optional `deny_message` on a deny is delivered to the agent as the rejected tool result so it can adjust its approach. A `user.tool_confirmation` for an event whose `evaluated_permission` is not `"ask"` is rejected with a 400 - that includes calls the server denied under `auto`; your client cannot override them.

拒绝时可选的 `deny_message` 会作为被拒的工具结果交付给代理，让它调整做法。对 `evaluated_permission` 不是 `"ask"` 的事件发送 `user.tool_confirmation` 会被 400 拒绝——包括服务器在 `auto` 下拒绝的调用；你的客户端无法推翻它们。

#### `auto` - let the server evaluate each call / `auto` —— 让服务器评估每次调用

Set `{"type": "auto"}` anywhere a `permission_policy` is accepted: a toolset's `default_config` or an individual `configs` entry, on the agent toolset or an `mcp_toolset`. Because the evaluation considers the call's input and the session's content up to that point, two calls to the same tool can be treated differently. Each call has exactly one of three outcomes:

在接受 `permission_policy` 的任何位置设置 `{"type": "auto"}`：工具集的 `default_config` 或单个 `configs` 条目，agent 工具集或 `mcp_toolset` 皆可。由于评估会考虑调用的输入和截至当时的会话内容，对同一工具的两次调用可能被区别对待。每次调用恰有三种结果之一：

| Outcome | What happens |
|---|---|
| **Runs** | Server determined the call is safe - executes as under `always_allow`, without reaching your client. |
| **Denied** | Server evaluated the call as high-risk - the tool does not run. The agent receives an error tool result (`Permission to use {tool_name} has been denied.`, `is_error: true`), the session **keeps running**, and your client cannot override the denial. |
| **Pauses** | Server reached no determination - the session pauses exactly as under `always_ask`; respond with `user.tool_confirmation`. |

| 结果 | 发生什么 |
|---|---|
| **运行** | 服务器认定调用安全——按 `always_allow` 一样执行，不经过你的客户端。 |
| **拒绝** | 服务器评估该调用为高风险——工具不运行。代理收到错误工具结果（`Permission to use {tool_name} has been denied.`，`is_error: true`），会话**继续运行**，你的客户端无法推翻该拒绝。 |
| **暂停** | 服务器未能作出判定——会话完全按 `always_ask` 暂停；用 `user.tool_confirmation` 响应。 |

```json
{
  "name": "Ops Agent",
  "model": "claude-opus-5-5",
  "mcp_servers": [{ "type": "url", "name": "github", "url": "https://mcp.example.com/github" }],
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "permission_policy": { "type": "auto" } },
      "configs": [{ "name": "bash", "permission_policy": { "type": "always_ask" } }]
    },
    {
      "type": "mcp_toolset",
      "mcp_server_name": "github",
      "default_config": { "permission_policy": { "type": "auto" } }
    }
  ]
}
```

Pass the same shape as an untyped dict / object literal / hash in Python, TypeScript, and Ruby. The typed SDKs (Go, Java, C#, PHP) need a generated type for the `auto` policy that ships with each SDK's release of the feature - until then, build the request in an untyped language or via cURL / `ant`. Python and TypeScript also only type-check `{"type": "auto"}` from the release that adds it (the wire API accepts it regardless).

在 Python、TypeScript 和 Ruby 中以无类型 dict / 对象字面量 / 哈希传入同样的形状。类型化 SDK（Go、Java、C#、PHP）需要随各 SDK 支持该特性的版本一起发布的 `auto` 策略生成类型——在此之前，请用无类型语言或通过 cURL / `ant` 构建请求。Python 和 TypeScript 也要到添加它的那个版本起才对 `{"type": "auto"}` 做类型检查（线上 API 无论如何都接受它）。

**What the evaluation trusts.** The server treats session content as material to assess, not instructions to follow. Text you post in `user.message` events (including end-user text you relay there) counts as *your intent* and can lead the server to allow a call it would otherwise deny - though some calls are evaluated as high-risk regardless. The same words in a tool result, a fetched webpage, an MCP server response, or a message between session threads carry no such weight. If you relay untrusted end-user input in `user.message`, the server reads it as your intent too and it can get a call allowed - put `always_ask` on the tools you would not let that end user run without review.

**评估信任什么。** 服务器把会话内容当作待评估的材料，而不是要遵循的指令。你在 `user.message` 事件中发布的文本（包括你转述到那里的终端用户文本）被视为*你的意图*，可能让服务器放行一个它本会拒绝的调用——不过有些调用无论如何都会被评估为高风险。同样的文字出现在工具结果、抓取的网页、MCP 服务器响应或会话线程之间的消息里则没有这种分量。如果你在 `user.message` 中转述不受信任的终端用户输入，服务器也会把它读作你的意图，它可能让某个调用被放行——对那些你不会让该终端用户未经审查就运行的工具，请加上 `always_ask`。

【评论】这段区分了"操作者意图"（`user.message`）与普通"会话内容"（工具结果、网页、MCP 响应），是针对提示词注入的信任分级设计；文档同时提醒转述不可信输入会把注入文本提升为操作者意图。

> **`auto` is not a human checkpoint.** A call the server determines to be safe runs before any person sees it, and its effects may not be reversible. If a person must review a tool's calls before they run, use `always_ask` on that tool.
> **`auto` 不是人工检查点。** 服务器认定安全的调用会在任何人看到之前就运行，其效果可能不可逆。如果必须有人在工具调用运行前审查，请对该工具使用 `always_ask`。

#### `evaluated_permission` and `evaluation` - see how each call was evaluated / `evaluated_permission` 与 `evaluation` —— 查看每次调用的评估结果

Under **any** policy, each `agent.tool_use` and `agent.mcp_tool_use` event carries `evaluated_permission` (`"allow" | "ask" | "deny"`) - the outcome of the permission check. Most events also carry an `evaluation` object whose `type` names the policy that produced the outcome; under `auto` it adds the server's determination and, for `ask` / `deny`, a `reason_code`:

在**任何**策略下，每个 `agent.tool_use` 和 `agent.mcp_tool_use` 事件都携带 `evaluated_permission`（`"allow" | "ask" | "deny"`）——即权限检查的结果。多数事件还携带一个 `evaluation` 对象，其 `type` 标明产生该结果的策略；在 `auto` 下它还包含服务器的判定，对 `ask` / `deny` 另有 `reason_code`：

```json
{
  "type": "agent.tool_use",
  "id": "sevt_01pqr...",
  "name": "bash",
  "input": { "command": "rm -rf /workspace/reports" },
  "evaluated_permission": "deny",
  "evaluation": {
    "type": "auto",
    "evaluated_permission": { "type": "deny", "reason_code": "high_risk" }
  },
  "processed_at": "2026-03-25T14:05:12Z"
}
```

| `evaluation` | Top-level `evaluated_permission` | Meaning |
|---|---|---|
| `{"type": "always_allow"}` | `"allow"` | Resolved policy is `always_allow`; the call ran. |
| `{"type": "always_ask"}` | `"ask"` | Resolved policy is `always_ask`; paused for your approval. |
| `{"type": "auto", "evaluated_permission": {"type": "allow"}}` | `"allow"` | Server determined the call safe; it ran. |
| `{"type": "auto", "evaluated_permission": {"type": "ask", "reason_code": "indeterminate"}}` | `"ask"` | Server reached no determination; paused for your approval. |
| `{"type": "auto", "evaluated_permission": {"type": "deny", "reason_code": "high_risk"}}` | `"deny"` | Server evaluated the call as high-risk and denied it. |

| `evaluation` | 顶层 `evaluated_permission` | 含义 |
|---|---|---|
| `{"type": "always_allow"}` | `"allow"` | 解析出的策略是 `always_allow`；调用已运行。 |
| `{"type": "always_ask"}` | `"ask"` | 解析出的策略是 `always_ask`；已暂停等待你的批准。 |
| `{"type": "auto", "evaluated_permission": {"type": "allow"}}` | `"allow"` | 服务器认定调用安全；已运行。 |
| `{"type": "auto", "evaluated_permission": {"type": "ask", "reason_code": "indeterminate"}}` | `"ask"` | 服务器未能作出判定；已暂停等待你的批准。 |
| `{"type": "auto", "evaluated_permission": {"type": "deny", "reason_code": "high_risk"}}` | `"deny"` | 服务器评估该调用为高风险并拒绝了它。 |

- On the `auto` form the nested `evaluated_permission.type` always equals the event's top-level `evaluated_permission`.
  在 `auto` 形式下，嵌套的 `evaluated_permission.type` 始终等于事件的顶层 `evaluated_permission`。
- `reason_code` is for your client to branch on and keep in audit records - not text to show end users.
  `reason_code` 供你的客户端做分支判断并存入审计记录——不是给终端用户看的文本。
- `evaluation` is **absent** when the agent names a tool that isn't enabled in the session (server denies without evaluating any policy: `evaluated_permission: "deny"`, no `evaluation`) and on events recorded before the field existed (read those as `always_allow` for `"allow"`, `always_ask` for `"ask"`).
  当代理指名一个会话中未启用的工具时，`evaluation` **不存在**（服务器不做任何策略评估直接拒绝：`evaluated_permission: "deny"`，无 `evaluation`）；在该字段出现之前记录的事件上也不存在（`"allow"` 读作 `always_allow`，`"ask"` 读作 `always_ask`）。
- Write your client to tolerate an `evaluation.type` or `reason_code` it doesn't recognize.
  写客户端时要能容忍它不认识的 `evaluation.type` 或 `reason_code`。
- `agent.custom_tool_use` events carry neither field (custom tools aren't governed by permission policies).
  `agent.custom_tool_use` 事件两个字段都不带（自定义工具不受权限策略管辖）。

To enable only specific tools, flip the default off and opt-in per tool:

要只启用特定工具，把默认关掉并按工具逐个选入：

```json
{
  "tools": [
    {
      "type": "agent_toolset_20260401",
      "default_config": { "enabled": false },
      "configs": [
        { "name": "bash", "enabled": true },
        { "name": "read", "enabled": true }
      ]
    }
  ]
}
```

### Web search & web fetch settings (domain filters) / 网络搜索与网络抓取设置（域名过滤器）

`web_search` and `web_fetch` run on Anthropic's servers regardless of environment type, so an environment's `networking` policy **does not** govern them (see `shared/managed-agents-environments.md` -> Networking). To control what they can reach, set `allowed_domains` (only these hosts) **or** `blocked_domains` (never these hosts) - never both on one entry - on the tool's `configs` entry. Each tool carries its own list. Organization-level web search/fetch settings in the Console apply to the Messages API only, not to Managed Agents sessions.

`web_search` 和 `web_fetch` 无论环境类型如何都运行在 Anthropic 的服务器上，因此环境的 `networking` 策略**不**管辖它们（见 `shared/managed-agents-environments.md` -> Networking）。要控制它们能访问什么，在工具的 `configs` 条目上设置 `allowed_domains`（只许这些主机）**或** `blocked_domains`（绝不去这些主机）——同一条目上绝不同时设置。每个工具携带自己的列表。Console 中组织级的网络搜索/抓取设置只适用于 Messages API，不适用于 Managed Agents 会话。

```json
{
  "type": "agent_toolset_20260401",
  "configs": [
    {
      "type": "web_search",
      "name": "web_search",
      "allowed_domains": ["docs.example.com", "arxiv.org"],
      "user_location": { "type": "approximate", "country": "US", "timezone": "America/Los_Angeles" }
    },
    {
      "type": "web_fetch",
      "name": "web_fetch",
      "blocked_domains": ["ads.example.com"],
      "max_content_tokens": 50000
    }
  ]
}
```

| Setting | Applies to | Description |
|---|---|---|
| `allowed_domains` | `web_search`, `web_fetch` | The only hosts the tool can reach. Mutually exclusive with `blocked_domains` on the same entry. |
| `blocked_domains` | `web_search`, `web_fetch` | Hosts the tool cannot reach. |
| `max_content_tokens` | `web_fetch` | Positive integer cap on fetched *text* content entering context (binary content such as PDFs is not capped). |
| `user_location` | `web_search` | `{ "type": "approximate", city?, region?, country? (2-letter uppercase ISO 3166-1), timezone? (IANA) }` - at least one of the optional fields. |

| 设置 | 适用于 | 说明 |
|---|---|---|
| `allowed_domains` | `web_search`、`web_fetch` | 工具唯一能访问的主机。与同一条目上的 `blocked_domains` 互斥。 |
| `blocked_domains` | `web_search`、`web_fetch` | 工具不能访问的主机。 |
| `max_content_tokens` | `web_fetch` | 进入上下文的被抓取*文本*内容的正整数上限（PDF 等二进制内容不受限）。 |
| `user_location` | `web_search` | `{ "type": "approximate", city?, region?, country? (2 字母大写 ISO 3166-1), timezone? (IANA) }`——可选字段至少填一个。 |

**Run-time behavior:** a `web_fetch` call outside its list returns an error result to the agent (`is_error: true` on `agent.tool_result`, content names `url_not_allowed`); `web_search` silently omits results outside its list. In the Console, the agent form has allow/block-list controls for the web tools; `user_location` and `max_content_tokens` are set in the agent's **Raw** view.

**运行时行为：** 列表之外的 `web_fetch` 调用会向代理返回错误结果（`agent.tool_result` 上 `is_error: true`，内容指名 `url_not_allowed`）；`web_search` 则静默省略列表之外的结果。在 Console 中，代理表单为网络工具提供允许/阻止列表控件；`user_location` 和 `max_content_tokens` 在代理的 **Raw** 视图中设置。

**Domain list rules** (violations -> 400 `invalid_request_error` on agent create/update and on session create/update that supplies `tools`; messages name the list and zero-based index, e.g. `allowed_domains.0: IP addresses are not supported...`):

**域名列表规则**（违反 -> 在代理创建/更新以及提供 `tools` 的会话创建/更新上返回 400 `invalid_request_error`；错误消息指名列表和从零起的下标，例如 `allowed_domains.0: IP addresses are not supported...`）：

- 1-64 domains per list, each 1-255 chars. Empty list is rejected - omit the field or send `null` for "no restriction". Duplicates within a list are rejected.
  每个列表 1-64 个域名，每个 1-255 字符。空列表被拒绝——省略该字段或发送 `null` 表示"无限制"。列表内重复项被拒绝。
- Plain hostname only: `example.com`, not `https://example.com`, `example.com:443`, or `*.example.com`. Case-insensitive; a single trailing `/` is ignored.
  仅限纯主机名：`example.com`，而不是 `https://example.com`、`example.com:443` 或 `*.example.com`。不区分大小写；单个尾部 `/` 被忽略。
- A listed domain covers itself **and its subdomains** (`example.com` covers `docs.example.com`; `docs.example.com` does not cover `example.com` or `api.example.com`). `www.` is an ordinary subdomain - list the bare domain to cover both.
  列出的域名覆盖其自身**及其子域**（`example.com` 覆盖 `docs.example.com`；`docs.example.com` 不覆盖 `example.com` 或 `api.example.com`）。`www.` 是普通子域——列出裸域名即可两者都覆盖。
- Rejected: IP addresses in any form; bare TLDs/registry suffixes (`com`, `co.uk`); single-label names (`intranet`); `localhost` and hosts ending in `.localhost`, `.local`, `.internal`, `.localdomain`, `.invalid`; non-ASCII (use `xn--` Punycode).
  被拒绝的：任何形式的 IP 地址；裸 TLD/注册局后缀（`com`、`co.uk`）；单标签名（`intranet`）；`localhost` 以及以 `.localhost`、`.local`、`.internal`、`.localdomain`、`.invalid` 结尾的主机；非 ASCII（请用 `xn--` Punycode）。
- `web_fetch` domains cannot carry a path. `web_search` domains may carry a path suffix (`example.com/blog`, no spaces / `?` / `#` / `$ , | ^ !`), but the provider matches it as a URL pattern - prefer plain hostnames.
  `web_fetch` 域名不能带路径。`web_search` 域名可以带路径后缀（`example.com/blog`，不含空格 / `?` / `#` / `$ , | ^ !`），但提供方把它当作 URL 模式匹配——优先使用纯主机名。
- Provider-dependent rejections at the same time: a domain Anthropic's crawler may not access, an unsupported `user_location.country` (message ends `not a country the search provider supports`), an invalid IANA `timezone`.
  与提供方相关的同刻拒绝：Anthropic 爬虫不可访问的域名、不受支持的 `user_location.country`（消息以 `not a country the search provider supports` 结尾）、无效的 IANA `timezone`。

The session re-checks the config when it first initializes the tool; if a previously accepted setting is no longer valid it emits `session.error` and goes `idle` without retrying. Fix via a session tools update (`shared/managed-agents-core.md` -> Updating the agent configuration mid-session), update the agent too so new sessions get the fix, then send a new `user.message`.

会话在首次初始化工具时会重新检查配置；如果先前被接受的设置不再有效，它会发出 `session.error` 并转为 `idle`，不做重试。通过会话工具更新来修复（`shared/managed-agents-core.md` -> Updating the agent configuration mid-session），同时更新代理让新会话获得修复，然后发送新的 `user.message`。

**Multiagent layering** (see `shared/managed-agents-multiagent.md`): every list on the path to a thread applies at once - a roster agent is bound by its own lists, by those of every agent that called it, and by the coordinator's *current* lists. Allow-lists intersect and block-lists union, so a roster agent can narrow but never widen. Disjoint allow-lists leave the tool available but every call fails `url_not_allowed` (the tool description tells the model) - keep roster allow-lists inside the coordinator's. `max_content_tokens` and `user_location` are **not** combined: own value -> caller's -> coordinator's. `{"type": "self"}` entries follow the coordinator. The outcome grader (`shared/managed-agents-outcomes.md`) runs without the web tools. Updating an idle session's tools changes the coordinator's lists for every thread from its next turn; a roster agent's own lists stay as defined at session create.

**多代理分层**（见 `shared/managed-agents-multiagent.md`）：通往线程的路径上每一份列表都同时生效——花名册代理同时受自己的列表、每个调用它的代理的列表、以及协调者*当前*列表的约束。允许列表取交集、阻止列表取并集，因此花名册代理只能收窄、绝不能放宽。互不相交的允许列表会让工具仍可用但每次调用都失败 `url_not_allowed`（工具描述会告诉模型）——把花名册允许列表保持在协调者的范围之内。`max_content_tokens` 和 `user_location` **不**合并：自己的值 -> 调用者的 -> 协调者的。`{"type": "self"}` 条目跟随协调者。结果评分器（`shared/managed-agents-outcomes.md`）在没有网络工具的情况下运行。更新空闲会话的工具会从下一个回合起改变协调者对所有线程的列表；花名册代理自己的列表保持会话创建时的定义。

**vs. the Messages API `web_search_20260209` / `web_fetch_20260209` tools:** same `allowed_domains` / `blocked_domains` vocabulary, but 64-entry cap, no path on `web_fetch` domains, and no `max_uses`, `citations`, or `cache_control`. If migrating from Messages API, these move from per-request to once-on-the-agent.

**与 Messages API 的 `web_search_20260209` / `web_fetch_20260209` 工具对比：** 同样的 `allowed_domains` / `blocked_domains` 词汇，但有 64 条上限、`web_fetch` 域名不带路径，且没有 `max_uses`、`citations` 或 `cache_control`。如果从 Messages API 迁移，这些设置从按请求传入变为在代理上设置一次。

### Custom Tools (Client-Side) / 自定义工具（客户端侧）

Custom tools are executed by **your application**, not Anthropic. The flow:

自定义工具由**你的应用**执行，而不是 Anthropic。流程如下：

1. Agent decides to use the tool -> session emits a `agent.custom_tool_use` event with inputs
   代理决定使用工具 -> 会话发出携带输入的 `agent.custom_tool_use` 事件
2. Session goes `idle` waiting for you
   会话转为 `idle` 等待你
3. Your application executes the tool
   你的应用执行该工具
4. You send back a `user.custom_tool_result` event with the output
   你回发携带输出的 `user.custom_tool_result` 事件
5. Session resumes `running`
   会话恢复 `running`

No permission policy needed - you're the one executing.

无需权限策略——执行者就是你自己。

```json
{
  "tools": [
    {
      "type": "custom",
      "name": "get_weather",
      "description": "Fetch current weather for a city.",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": { "type": "string", "description": "City name" }
        },
        "required": ["city"]
      }
    }
  ]
}
```

### MCP Servers / MCP 服务器

MCP (Model Context Protocol) servers expose standardized third-party capabilities (e.g. Asana, GitHub, Linear). **Configuration is split across agent and vault:**

MCP（Model Context Protocol）服务器暴露标准化的第三方能力（如 Asana、GitHub、Linear）。**配置拆分在代理和保管库两处：**

1. **Agent creation** declares which servers to connect to (`type`, `name`, `url` - no auth). The agent's `mcp_servers` array has no auth field.
   **代理创建**声明要连接哪些服务器（`type`、`name`、`url`——不含认证）。代理的 `mcp_servers` 数组没有认证字段。
2. **Vault** stores the OAuth credentials. Attach via `vault_ids` on session create.
   **保管库**存储 OAuth 凭证。在会话创建时通过 `vault_ids` 附加。

This keeps secrets out of reusable agent definitions. Each vault credential is tied to one MCP server URL; Anthropic matches credentials to servers by URL.

这样就把机密挡在可复用的代理定义之外。每个保管库凭证绑定一个 MCP 服务器 URL；Anthropic 按 URL 把凭证与服务器配对。

**Agent side - declare servers (no auth):**

**代理侧——声明服务器（不含认证）：**

| Field | Required | Description |
|---|---|---|
| `type` | Yes | `"url"` |
| `name` | Yes | Unique name - referenced by `mcp_toolset.mcp_server_name` |
| `url` | Yes | The MCP server's endpoint URL (Streamable HTTP transport) |

| 字段 | 必需 | 说明 |
|---|---|---|
| `type` | 是 | `"url"` |
| `name` | 是 | 唯一名称——由 `mcp_toolset.mcp_server_name` 引用 |
| `url` | 是 | MCP 服务器的端点 URL（Streamable HTTP 传输） |

```json
{
  "mcp_servers": [
    { "type": "url", "name": "linear", "url": "https://mcp.linear.app/mcp" }
  ],
  "tools": [
    { "type": "mcp_toolset", "mcp_server_name": "linear" }
  ]
}
```

**Session side - attach vault:**

**会话侧——附加保管库：**

```json
{
  "agent": "agent_abc123",
  "environment_id": "env_abc123",
  "vault_ids": ["vlt_abc123"]
}
```

> Tip: **Per-tool enablement:** `mcp_toolset` accepts `default_config: {enabled: false}` + `configs: [{name, enabled: true}]` for an allowlist pattern. MCP `configs` entries take **only** `name` (the bare tool name as the server reports it), `enabled`, and `permission_policy` - no `type` field and none of the web settings that `web_search` / `web_fetch` accept in the agent toolset.
> 提示：**按工具启用：** `mcp_toolset` 接受 `default_config: {enabled: false}` + `configs: [{name, enabled: true}]` 实现允许清单模式。MCP 的 `configs` 条目**只**接受 `name`（服务器报告的裸工具名）、`enabled` 和 `permission_policy`——没有 `type` 字段，也没有 `web_search` / `web_fetch` 在 agent 工具集中接受的那些 web 设置。

> Tip: **Changing tools/MCP servers on a running session:** `sessions.update()` can replace `agent.tools` and `agent.mcp_servers` while the session is `idle` - a session-local override that doesn't touch the agent object. `vault_ids` is create-only. See `shared/managed-agents-core.md` -> Updating the agent configuration mid-session.
> 提示：**在运行中的会话上更换工具/MCP 服务器：** 会话处于 `idle` 时，`sessions.update()` 可以替换 `agent.tools` 和 `agent.mcp_servers`——一种不触碰代理对象的会话本地覆盖。`vault_ids` 仅创建时可用。见 `shared/managed-agents-core.md` -> Updating the agent configuration mid-session。

**Large tool outputs.** If a tool returns more than **100,000 characters (roughly 25,000 tokens)**, the output is automatically offloaded to a file in the sandbox - the agent receives a truncated preview plus the file path and can `read` the full content. No configuration required. The threshold is in *characters*, not tokens, and applies to built-in agent tools as well as MCP tools.

**大型工具输出。** 如果工具返回超过 **100,000 字符（约 25,000 token）**，输出会自动卸载到沙箱中的一个文件——代理收到截断的预览加文件路径，并可用 `read` 读取完整内容。无需配置。阈值以*字符*计而非 token，对内置 agent 工具和 MCP 工具同样适用。

**Invalid vault credentials don't block session creation.** If a vault credential is invalid for a declared MCP server, the session still creates successfully; a `session.error` event describes the MCP auth failure, and auth retries on the next `session.status_idle` -> `session.status_running` transition.

**无效的保管库凭证不会阻止会话创建。** 如果某个保管库凭证对已声明的 MCP 服务器无效，会话仍会创建成功；`session.error` 事件描述该 MCP 认证失败，认证会在下一次 `session.status_idle` -> `session.status_running` 转换时重试。

> Warning: **MCP auth tokens != REST API tokens.** Hosted MCP servers (`mcp.notion.com`, `mcp.linear.app`, etc.) typically require **OAuth bearer tokens**, not the service's native API keys. A Notion `ntn_` integration token authenticates against Notion's REST API but will **not** work as a vault credential for the Notion MCP server. These are different auth systems.
> 警告：**MCP 认证 token != REST API token。** 托管的 MCP 服务器（`mcp.notion.com`、`mcp.linear.app` 等）通常要求 **OAuth bearer token**，而不是该服务的原生 API key。Notion 的 `ntn_` integration token 能对 Notion 的 REST API 认证，但**不能**作为 Notion MCP 服务器的保管库凭证。这是两套不同的认证体系。

### Vaults - the credential store / 保管库——凭证存储

**Vaults** store credentials that Anthropic manages on your behalf. Two credential categories:

**保管库**存储由 Anthropic 代你管理的凭证。两类凭证：

- **MCP credentials** (`mcp_oauth`, `static_bearer`) - keyed by `mcp_server_url`. When the agent connects to a server at that URL, the token is injected automatically. **Matching is normalized, not byte-exact:** scheme and host are lowercased, and default ports and trailing slashes are stripped, so host casing, an explicit default port, or a trailing slash won't break the match. A different path, subdomain, or *non-default* port will. If nothing matches, the connection is attempted unauthenticated. `mcp_oauth` tokens are auto-refreshed via the standard OAuth 2.0 `refresh_token` grant. This is the only way to authenticate MCP servers.
  **MCP 凭证**（`mcp_oauth`、`static_bearer`）——以 `mcp_server_url` 为键。代理连接该 URL 上的服务器时，token 被自动注入。**匹配是归一化的，不是逐字节精确：** scheme 和 host 转为小写，默认端口和尾部斜杠被剥除，因此 host 大小写、显式默认端口或尾部斜杠都不会破坏匹配。不同的路径、子域或*非默认*端口则会。若无匹配，连接将以未认证方式尝试。`mcp_oauth` token 通过标准 OAuth 2.0 `refresh_token` 授权自动刷新。这是认证 MCP 服务器的唯一方式。
- **Environment variables** (`environment_variable`) - keyed by `secret_name` (the env var name). The sandbox sees only an **opaque placeholder**; the real secret is substituted into the outbound request **at egress**. Use this for any service that authenticates through an environment variable: CLIs (`aws`, `gcloud`, `stripe`), SDKs, or direct `curl` calls from the `bash` tool.
  **环境变量**（`environment_variable`）——以 `secret_name`（环境变量名）为键。沙箱只看到一个**不透明占位符**；真实机密在**出口处**替换进外发请求。任何通过环境变量认证的服务都用它：CLI（`aws`、`gcloud`、`stripe`）、SDK，或从 `bash` 工具直接发起的 `curl` 调用。

Secret fields you supply (`token`, `access_token`, `refresh_token`, `client_secret`, `secret_value`) are write-only - never returned in API responses.

你提供的机密字段（`token`、`access_token`、`refresh_token`、`client_secret`、`secret_value`）是只写的——绝不出现在 API 响应中。

#### Credentials and the sandbox / 凭证与沙箱

Vaults store credentials; those credentials **never enter the sandbox**. This is a deliberate security boundary - code running in the sandbox (including anything the agent writes) cannot read or exfiltrate a vaulted credential, even under prompt injection. Instead, credentials are injected by Anthropic-side proxies **after** a request leaves the sandbox:

保管库存储凭证；这些凭证**从不进入沙箱**。这是一道刻意设置的安全边界——沙箱中运行的代码（包括代理写下的一切）无法读取或外泄保管库中的凭证，即使遭遇提示词注入也一样。替代做法是，凭证由 Anthropic 侧的代理在请求**离开沙箱之后**注入：

【评论】"凭证不进沙箱、出口处注入、路径机密不可入库"是明确的防外泄设计；文档把提示词注入列为该边界需要抵御的威胁模型之一。

- **MCP tool calls** are routed through an Anthropic-side proxy that fetches the credential from the vault and adds it to the outbound request.
  **MCP 工具调用**经过一个 Anthropic 侧的代理路由，该代理从保管库取凭证并加到外发请求上。
- **Git operations on attached GitHub repositories** (`git pull`, `git push`, GitHub REST calls) are routed through a git proxy that injects the `github_repository` resource's `authorization_token` the same way.
  **对已附加 GitHub 仓库的 Git 操作**（`git pull`、`git push`、GitHub REST 调用）经过一个 git 代理路由，以同样方式注入 `github_repository` 资源的 `authorization_token`。
- **Environment-variable credentials** appear in the sandbox as an opaque placeholder; the real value replaces the placeholder at egress, on requests to the credential's allowed hosts only. Substitution covers request **headers and body only** - a secret embedded in the **URL path** is never substituted, so path-secret endpoints (e.g. Slack incoming-webhook URLs) can't be vaulted; use header-based auth instead (for Slack: a bot token in `Authorization` via `chat.postMessage`).
  **环境变量凭证**在沙箱中表现为不透明占位符；真实值在出口处替换占位符，且只对发往该凭证允许主机的请求。替换只覆盖请求**头和体**——嵌入 **URL 路径**的机密绝不被替换，因此路径机密端点（如 Slack incoming-webhook URL）无法入库；请改用基于请求头的认证（Slack：通过 `chat.postMessage` 在 `Authorization` 中放 bot token）。

**When vault credentials don't fit** (e.g. self-hosted sandboxes - `environment_variable` is not yet supported there), **register a custom tool:** the agent emits `agent.custom_tool_use`, your orchestrator (which already holds the credential) executes the call and returns `user.custom_tool_result` over the same authenticated event stream. No public endpoint is exposed; the sandbox never sees the secret. See `shared/managed-agents-client-patterns.md` -> Pattern 9.

**当保管库凭证不适用时**（例如自托管沙箱——那里尚不支持 `environment_variable`），**注册一个自定义工具：** 代理发出 `agent.custom_tool_use`，你的编排器（已持有凭证）执行调用并通过同一认证事件流返回 `user.custom_tool_result`。不暴露任何公开端点；沙箱永远看不到机密。见 `shared/managed-agents-client-patterns.md` -> Pattern 9。

**Do not put API keys in the system prompt or user messages as a workaround** - they persist in the session's event history.

**不要把 API key 放进系统提示词或用户消息里当作变通手段**——它们会持久留在会话的事件历史中。

> Formerly known internally as TATs (Tool/Tenant Access Tokens).
> 内部曾称为 TAT（Tool/Tenant Access Tokens）。

**Flow:**

**流程：**

1. Create a vault (`client.beta.vaults.create(...)`) - one per tenant/user, or one shared, depending on your model
   创建保管库（`client.beta.vaults.create(...)`）——每个租户/用户一个，或共享一个，取决于你的模型
2. Add credentials to it (`client.beta.vaults.credentials.create(...)`) - MCP credentials are keyed by MCP server URL; environment-variable credentials by `secret_name`
   向其添加凭证（`client.beta.vaults.credentials.create(...)`）——MCP 凭证以 MCP 服务器 URL 为键；环境变量凭证以 `secret_name` 为键
3. Reference the vault on session create via `vault_ids: ["vlt_..."]`
   在会话创建时通过 `vault_ids: ["vlt_..."]` 引用保管库
4. Anthropic auto-refreshes OAuth tokens before they expire and substitutes secrets at runtime
   Anthropic 在 OAuth token 过期前自动刷新，并在运行时替换机密

**MCP OAuth credential shape**:

**MCP OAuth 凭证形状**：

```json
{
  "display_name": "Notion (workspace-foo)",
  "auth": {
    "type": "mcp_oauth",
    "mcp_server_url": "https://mcp.notion.com/mcp",
    "access_token": "<current access token>",
    "expires_at": "2026-04-02T14:00:00Z",
    "refresh": {
      "refresh_token": "<refresh token>",
      "client_id": "<your OAuth client_id>",
      "token_endpoint": "https://api.notion.com/v1/oauth/token",
      "token_endpoint_auth": { "type": "none" }
    }
  }
}
```

The `refresh` block is what enables auto-refresh - `token_endpoint` is where Anthropic posts the `refresh_token` grant. `token_endpoint_auth` is a discriminated union:

`refresh` 块是自动刷新得以工作的关键——`token_endpoint` 是 Anthropic 提交 `refresh_token` 授权的地方。`token_endpoint_auth` 是一个可判别联合：

| `type` | Shape | Use when |
|---|---|---|
| `"none"` | `{type: "none"}` | Public OAuth client (no secret) |
| `"client_secret_basic"` | `{type: "client_secret_basic", client_secret: "..."}` | Confidential client, secret via HTTP Basic auth |
| `"client_secret_post"` | `{type: "client_secret_post", client_secret: "..."}` | Confidential client, secret in request body |

| `type` | 形状 | 适用场景 |
|---|---|---|
| `"none"` | `{type: "none"}` | 公开 OAuth 客户端（无密钥） |
| `"client_secret_basic"` | `{type: "client_secret_basic", client_secret: "..."}` | 机密客户端，密钥经 HTTP Basic 认证 |
| `"client_secret_post"` | `{type: "client_secret_post", client_secret: "..."}` | 机密客户端，密钥放在请求体中 |

Omit `refresh` entirely if you only have an access token with no refresh capability - it'll work until it expires, then the agent loses access.

如果只有一个无刷新能力的 access token，就整个省略 `refresh`——它能用到过期为止，之后代理失去访问权。

> Tip: **Getting an OAuth token.** How you obtain the initial access and refresh tokens depends on the MCP server - consult its documentation. Once you have them, store them in a vault credential using the shape above; Anthropic auto-refreshes via the `refresh.token_endpoint` from there.
> 提示：**获取 OAuth token。** 初始 access 和 refresh token 的获取方式取决于 MCP 服务器——请查阅其文档。拿到后，用上面的形状存入保管库凭证；此后 Anthropic 经 `refresh.token_endpoint` 自动刷新。

**Environment-variable credential shape**:

**环境变量凭证形状**：

```json
{
  "display_name": "Twilio API key for sandbox",
  "auth": {
    "type": "environment_variable",
    "secret_name": "TWILIO_API_KEY",
    "secret_value": "sk-your-secret-here",
    "networking": {
      "type": "limited",
      "allowed_hosts": ["api.twilio.com", "*.twilio.com"]
    }
  }
}
```

`networking.allowed_hosts` controls which outbound hosts the secret can be substituted for - `{"type": "limited", "allowed_hosts": [...]}` or `{"type": "unrestricted"}` if you can't enumerate the domains in advance. Limiting is strongly recommended: it prevents the key from ever being sent to unauthorized hosts.

`networking.allowed_hosts` 控制机密可以被替换进哪些外发主机的请求——`{"type": "limited", "allowed_hosts": [...]}`，若无法预先枚举域名则用 `{"type": "unrestricted"}`。强烈建议限制：它防止 key 被发送到未授权主机。

**`injection_location`** (optional, sibling of `networking`) controls **where** in the outbound request the secret is substituted - `{header: bool, body: bool}`. The two are independent: `allowed_hosts` scopes *which hosts* a substituted request can target; `injection_location` scopes *which parts of the request* the secret is substituted into across all of those hosts. Most services read an API key from a request header, so `{"header": true}` is the narrower configuration - request bodies are often assembled from content the agent is working with, making the body the broader exposure surface. A placeholder in a disabled location is **neither substituted nor stripped** - the literal opaque placeholder string is sent to the third party in that location.

**`injection_location`**（可选，`networking` 的兄弟字段）控制机密被替换进外发请求的**哪个位置**——`{header: bool, body: bool}`。两者相互独立：`allowed_hosts` 限定被替换的请求可以以*哪些主机*为目标；`injection_location` 限定在所有这些主机上机密被替换进*请求的哪些部分*。多数服务从请求头读取 API key，因此 `{"header": true}` 是更窄的配置——请求体常由代理正在处理的内容拼装而成，使请求体成为更宽的暴露面。被禁用位置中的占位符**既不替换也不剔除**——字面的不透明占位符字符串会在那个位置原样发给第三方。

| Operation | `injection_location` semantics |
|---|---|
| Create credential | Omit the field entirely -> both locations enabled. Provide the object -> any field you omit defaults to `false` (`{"header": true}` creates a header-only credential). |
| Update credential | Fields **merge individually** - `{"body": false}` disables body substitution and leaves `header` unchanged. For a running session, the update takes effect on the session's next operation. |

| 操作 | `injection_location` 语义 |
|---|---|
| 创建凭证 | 完全省略该字段 -> 两个位置都启用。提供该对象 -> 省略的任何字段默认为 `false`（`{"header": true}` 创建仅请求头的凭证）。 |
| 更新凭证 | 字段**逐个合并**——`{"body": false}` 禁用体替换且 `header` 不变。对运行中的会话，更新在该会话的下一次操作时生效。 |

A credential must have at least one location enabled; a create or update that would disable both returns 400, as does explicit `null` for the object or either field (omit instead). The response always returns both fields with their resolved values.

凭证必须至少启用一个位置；会把两者都禁用的创建或更新返回 400，对对象或任一字段显式给 `null` 也一样（应改为省略）。响应始终返回两个字段及其解析后的值。

> Warning: **Credentials created in the Console are header-only by default** - unlike the API, where omitting the field enables both. If your client sends the secret in the request body (a form-encoded token request, for example), the placeholder passes through literally and the service rejects it with its own authentication error. Tick body injection in the Console form, or `POST` the credential with `{"injection_location": {"body": true}}`.
> 警告：**在 Console 中创建的凭证默认仅请求头**——与 API 不同，API 中省略该字段即启用两者。如果你的客户端在请求体中发送机密（例如表单编码的 token 请求），占位符会原样通过，服务会以它自己的认证错误拒绝。请在 Console 表单中勾选体注入，或用 `{"injection_location": {"body": true}}` `POST` 该凭证。

> Warning: **Two networking layers, both required.** `networking.allowed_hosts` on the credential controls which requests *use the secret*, not which requests are *allowed*. The agent must also be able to reach the domain at the **environment level** (`unrestricted`, or the host listed in the environment's `allowed_hosts` - see `shared/managed-agents-environments.md`). A domain missing from either layer means the secret-substituted request fails.
> 警告：**两层网络控制，缺一不可。** 凭证上的 `networking.allowed_hosts` 控制哪些请求*使用该机密*，而不是哪些请求*被允许*。代理还必须能在**环境层**访问该域名（`unrestricted`，或环境 `allowed_hosts` 中列出的主机——见 `shared/managed-agents-environments.md`）。任何一层缺少该域名都意味着替换了机密的请求会失败。

> Warning: **Client-side validation caveat.** Substitution happens at egress, not inside the sandbox - clients that validate the credential *format* locally before making a network request (e.g. a CLI that checks the key starts with `sk-`) will see the opaque placeholder and may fail at startup. If a client rejects the credential before any network call, that's why.
> 警告：**客户端校验注意事项。** 替换发生在出口，而不是沙箱内——在发起网络请求前于本地校验凭证*格式*的客户端（例如检查 key 是否以 `sk-` 开头的 CLI）会看到不透明占位符，可能在启动时失败。如果客户端在发起任何网络调用之前就拒绝该凭证，原因就在这里。

> Tip: **Scope the key minimally.** The agent can do anything the key allows; a key with broader permissions than the task needs increases the blast radius if the agent behaves unexpectedly.
> 提示：**最小范围授权。** 代理能做该 key 允许的任何事；key 的权限超出任务所需会扩大代理行为失常时的波及范围。

**Not supported with self-hosted sandboxes** - `environment_variable` credentials require Anthropic-managed egress. See `shared/managed-agents-self-hosted-sandboxes.md`.

**自托管沙箱不支持**——`environment_variable` 凭证需要 Anthropic 管理的出口。见 `shared/managed-agents-self-hosted-sandboxes.md`。

**Constraints (all credential types):**

**约束（所有凭证类型）：**

- **Unique key per vault.** `mcp_server_url` (MCP credentials) and `secret_name` (environment-variable credentials) must be unique among active credentials in a vault; duplicates return a 409.
  **每个保管库内键唯一。** `mcp_server_url`（MCP 凭证）和 `secret_name`（环境变量凭证）在保管库的活跃凭证中必须唯一；重复返回 409。
- **Keys are immutable.** Secret values, `display_name`, and (on environment-variable credentials) `injection_location` can be updated; to change `mcp_server_url`, `secret_name`, `token_endpoint`, or `client_id`, archive the credential and create a new one. Archiving purges the secret and frees the key for a replacement.
  **键不可变。** 机密值、`display_name` 以及（环境变量凭证上的）`injection_location` 可以更新；要改 `mcp_server_url`、`secret_name`、`token_endpoint` 或 `client_id`，需归档该凭证并新建一个。归档会清除机密并释放键供替换。
- **Maximum 20 credentials per vault.**
  **每个保管库最多 20 个凭证。**
- Credentials are stored as provided and **not validated until session runtime** - an invalid credential surfaces as an authentication or downstream error during the session, which is emitted but does not block the session from continuing.
  凭证按提供的样子存储，**直到会话运行时才校验**——无效凭证在会话期间表现为认证或下游错误，会发出事件但不阻止会话继续。

**Scoping:** Vaults are workspace-scoped. Anyone with developer+ role in the API workspace can create, read (metadata only - secrets are write-only), and attach vaults. `vault_ids` can be set at session **create** time but not via session update (the SDK docstring says "Not yet supported; requests setting this field are rejected").

**范围：** 保管库按工作区划定。在 API 工作区拥有 developer+ 角色的任何人都可创建、读取（仅元数据——机密只写）和附加保管库。`vault_ids` 可在会话**创建**时设置，但不能通过会话更新设置（SDK 文档字符串说 "Not yet supported; requests setting this field are rejected"）。

---

## Skills / 技能

Skills are reusable, filesystem-based resources that provide your agent with domain-specific expertise: workflows, context, and best practices that transform general-purpose agents into specialists. Unlike prompts (conversation-level instructions for one-off tasks), skills load on-demand and eliminate the need to repeatedly provide the same guidance across multiple conversations.

技能是可复用的、基于文件系统的资源，为你的代理提供领域专长：工作流、上下文和最佳实践，把通用代理变成专家。与提示词（针对一次性任务的会话级指令）不同，技能按需加载，免去了在多次对话中反复提供同样指引的需要。

Skills reach the agent two ways: **attached** through the agent's `skills` array, or **loaded from a GitHub repository** mounted on the session (see § Skills from a GitHub repository below). The agent automatically uses them when relevant to the task at hand:

技能以两种方式到达代理：通过代理的 `skills` 数组**附加**，或从挂载到会话上的 GitHub 仓库**加载**（见下文 § Skills from a GitHub repository）。当与手头任务相关时，代理会自动使用它们：

| Type | What it is |
|---|---|
| **Pre-built Anthropic skills** | Common document tasks (PowerPoint, Excel, Word, PDF). Reference by name (e.g. `xlsx`). |
| **Custom skills** | Skills you've created in your organization via the Skills API. Reference by `skill_id` + optional `version`. |

| 类型 | 是什么 |
|---|---|
| **预构建 Anthropic 技能** | 常见文档任务（PowerPoint、Excel、Word、PDF）。按名称引用（如 `xlsx`）。 |
| **自定义技能** | 你在组织中通过 Skills API 创建的技能。按 `skill_id` + 可选 `version` 引用。 |

**Max 20 skills per agent.** Agent creation uses `managed-agents-2026-04-01`; the separate Skills API (for managing custom skill definitions) is out of beta and needs no beta header.

**每个代理最多 20 个技能。** 代理创建使用 `managed-agents-2026-04-01`；单独的 Skills API（管理自定义技能定义）已结束测试期，无需 beta 请求头。

### Enabling skills on a session / 在会话上启用技能

Skills are attached to the **agent** definition via `agents.create()`:

技能通过 `agents.create()` 附加到**代理**定义上：

```ts
const agent = await client.beta.agents.create(
  {
    name: "Financial Agent",
    model: "claude-opus-5-5",
    system: "You are a financial analysis agent.",
    skills: [
      { type: "anthropic", skill_id: "xlsx" },
      { type: "custom", skill_id: "skill_abc123", version: "latest" },
    ],
  }
);
```

Python:

Python：

```python
agent = client.beta.agents.create(
    name="Financial Agent",
    model="claude-opus-5-5",
    system="You are a financial analysis agent.",
    skills=[
        {"type": "anthropic", "skill_id": "xlsx"},
        {"type": "custom", "skill_id": "skill_abc123", "version": "latest"},
    ]
)
```

**Skill reference fields:**

**技能引用字段：**

| Field | Anthropic skill | Custom skill |
|---|---|---|
| `type` | `"anthropic"` | `"custom"` |
| `skill_id` | Skill name (e.g. `"xlsx"`, `"docx"`, `"pptx"`, `"pdf"`) | Skill ID from Skills API (e.g. `"skill_abc123"`) |
| `version` | `"latest"` or a specific version number | `"latest"` or a specific version number |

| 字段 | Anthropic 技能 | 自定义技能 |
|---|---|---|
| `type` | `"anthropic"` | `"custom"` |
| `skill_id` | 技能名称（如 `"xlsx"`、`"docx"`、`"pptx"`、`"pdf"`） | Skills API 的技能 ID（如 `"skill_abc123"`） |
| `version` | `"latest"` 或具体版本号 | `"latest"` 或具体版本号 |

`version` is optional on **both** kinds and defaults to `"latest"` - it is not custom-skill-only.

`version` 对**两种**技能都可省略，默认为 `"latest"`——并非只有自定义技能才有。

### Skills from a GitHub repository / 来自 GitHub 仓库的技能

Skills can also live in your codebase. When a session mounts a repository via the `github_repository` resource (see `shared/managed-agents-environments.md` -> GitHub Repositories), the repository's root `.claude/skills` directory is scanned at session start, and each skill found becomes available to the agent: it sees each discovered skill's name, description, and sandbox path, and reads the skill's `SKILL.md` (plus any scripts/resources it ships) when a task matches.

技能也可以放在你的代码库里。当会话通过 `github_repository` 资源挂载一个仓库时（见 `shared/managed-agents-environments.md` -> GitHub Repositories），仓库根部的 `.claude/skills` 目录会在会话开始时被扫描，找到的每个技能都对代理可用：它看到每个被发现技能的名称、描述和沙箱路径，并在任务匹配时读取该技能的 `SKILL.md`（及其附带的任何脚本/资源）。

**The agent can discover any skill in `.claude/skills/<skill-name>/`** - one directory level deep at the repository root. Skills in the following locations are not discoverable: a bare `.claude/skills/SKILL.md` (no skill directory), anything nested deeper (`.claude/skills/tools/code-review/SKILL.md`), a `skills/` directory outside `.claude`, or a `.claude/skills` inside a package subdirectory (though those can still surface when the agent reads files under that subtree). The `SKILL.md` format is the same as uploaded custom skills.

**代理可以发现 `.claude/skills/<skill-name>/` 中的任何技能**——仓库根部向下一层目录。以下位置的技能不可被发现：裸的 `.claude/skills/SKILL.md`（没有技能目录）、嵌套更深的任何内容（`.claude/skills/tools/code-review/SKILL.md`）、`.claude` 之外的 `skills/` 目录，或包子目录内的 `.claude/skills`（不过当代理读取该子树下的文件时，这些技能仍可能浮现）。`SKILL.md` 格式与上传的自定义技能相同。

> Warning: **Repository skills are agent instructions - treat them as part of your trust boundary.** Anyone who can commit to a mounted repository (a merged external PR, a compromised dependency, a contributor) can add or edit `.claude/skills/` content, and the platform loads it at session start with no review step - where session tools like `bash` and `web_fetch` give injected instructions real capability. Only mount repositories you trust, and audit `.claude/skills/` before mounting one with external contributors.
> 警告：**仓库技能就是代理指令——把它们当作你信任边界的一部分。** 任何能向已挂载仓库提交的人（被合并的外部 PR、被攻陷的依赖、某位贡献者）都可以添加或编辑 `.claude/skills/` 内容，而平台在会话开始时加载它、没有任何审查步骤——此时 `bash` 和 `web_fetch` 之类的会话工具会让被注入的指令具备真实能力。只挂载你信任的仓库，并在挂载有外部贡献者的仓库之前审计 `.claude/skills/`。

Rules:

规则：

- **Cloud sandboxes only** - self-hosted sandboxes don't support `github_repository` resources, so they can't load repository skills.
  **仅限云沙箱**——自托管沙箱不支持 `github_repository` 资源，因此无法加载仓库技能。
- **Scanned once, at session start**, from the repository state checked out then (the resource's `checkout` branch/commit, else the default branch). Commits pushed mid-session are not picked up - start a new session for updated skills. Repositories added to a *running* session are not scanned either.
  **只在会话开始时扫描一次**，基于当时检出的仓库状态（资源的 `checkout` 分支/提交，否则为默认分支）。会话中途推送的提交不会被采纳——要获取更新的技能请新开会话。加入*运行中*会话的仓库也不会被扫描。
- **Coexists with attached skills.** If a repository skill shares a name with an attached skill (or a skill from another mounted repo), both are available, each announced with its own path.
  **与附加技能共存。** 如果仓库技能与某个附加技能（或另一个挂载仓库的技能）同名，两者都可用，各自以自己的路径宣告。

### Skills API / Skills API

| Operation             | Method   | Path                                            |
| --------------------- | -------- | ----------------------------------------------- |
| Create Skill          | `POST`   | `/v1/skills`                                    |
| List Skills           | `GET`    | `/v1/skills`                                    |
| Get Skill             | `GET`    | `/v1/skills/{id}`                               |
| Delete Skill          | `DELETE` | `/v1/skills/{id}`                               |
| Create Version        | `POST`   | `/v1/skills/{id}/versions`                      |
| List Versions         | `GET`    | `/v1/skills/{id}/versions`                      |
| Get Version           | `GET`    | `/v1/skills/{id}/versions/{version}`            |
| Delete Version        | `DELETE` | `/v1/skills/{id}/versions/{version}`            |

| 操作 | 方法 | 路径 |
| --------------------- | -------- | ----------------------------------------------- |
| 创建技能          | `POST`   | `/v1/skills`                                    |
| 列出技能           | `GET`    | `/v1/skills`                                    |
| 获取技能             | `GET`    | `/v1/skills/{id}`                               |
| 删除技能          | `DELETE` | `/v1/skills/{id}`                               |
| 创建版本        | `POST`   | `/v1/skills/{id}/versions`                      |
| 列出版本         | `GET`    | `/v1/skills/{id}/versions`                      |
| 获取版本           | `GET`    | `/v1/skills/{id}/versions/{version}`            |
| 删除版本        | `DELETE` | `/v1/skills/{id}/versions/{version}`            |
