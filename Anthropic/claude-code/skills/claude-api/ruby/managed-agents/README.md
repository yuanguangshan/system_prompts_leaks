<!-- BILINGUAL-EN-ZH -->
# Managed Agents - Ruby / Managed Agents - Ruby

> **Bindings not shown here:** This README covers the most common managed-agents flows for Ruby. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the Ruby SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **此处未展示的绑定：**本 README 覆盖 Ruby 最常见的 managed-agents 流程。如果需要的类、方法、命名空间、字段或行为未在此展示，请按 `shared/live-sources.md` WebFetch Ruby SDK 仓库**或相关文档页**，而不要猜测。不要从 cURL 形态或其他语言的 SDK 外推。

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `client.beta.agents.create` and pass it to every subsequent `client.beta.sessions.create`; do not call `agents.create` in the request path. **Recommended:** define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

> **智能体是持久的——创建一次，按 ID 引用。**保存 `client.beta.agents.create` 返回的智能体 ID，并在之后每次 `client.beta.sessions.create` 时传入；不要在请求路径中调用 `agents.create`。**推荐做法：**把智能体与环境定义为纳入版本控制的文件，用 `ant apply` 同步——见 `shared/anthropic-cli.md`（其实时文档 URL 在 `shared/live-sources.md` 中）。CLI 负责控制平面（创建/更新）；你的代码负责数据平面（用已保存 ID 发起的会话）。以下示例展示的是必须以编程方式供应时的代码内创建；在生产环境中，创建调用应放在 setup 里，而不是请求路径中。

【评论】"控制平面 / 数据平面"的划分把资源生命周期管理与请求时路径解耦，同时明确告诫不要在请求路径中调用创建类接口——这是控制 API 调用量与状态漂移的常见实践。

## Installation / 安装

```bash
gem install anthropic
```

## Client Initialization / 客户端初始化

```ruby
require "anthropic"

# Default (uses ANTHROPIC_API_KEY env var)
client = Anthropic::Client.new

# Explicit API key
client = Anthropic::Client.new(api_key: "your-api-key")
```

> Warning: **Trailing underscores:** The Ruby SDK uses `system_:` and `send_(` (trailing underscore) to avoid shadowing `Kernel#system` and `Kernel#send`. Use these forms throughout managed-agents code.

> 警告：**尾随下划线：**Ruby SDK 使用 `system_:` 与 `send_(`（尾随下划线）来避免遮蔽 `Kernel#system` 与 `Kernel#send`。在 managed-agents 代码中一律使用这些形式。

---

## Create an Environment / 创建环境

```ruby
environment = client.beta.environments.create(
  name: "my-dev-env",
  config: {
    type: "cloud",
    networking: {type: "unrestricted"}
  }
)
puts "Environment ID: #{environment.id}" # env_...
```

---

## Create an Agent (required first step) / 创建智能体（必需的第一步）

> Warning: **There is no inline agent config.** `model`/`system_`/`tools` live on the agent object, not the session. Always start with `client.beta.agents.create()` - the session takes either `agent: agent.id` or the typed hash form `agent: {type: "agent", id: agent.id, version: agent.version}`.

> 警告：**没有内联的智能体配置。**`model`/`system_`/`tools` 位于智能体对象上，而非会话上。始终从 `client.beta.agents.create()` 开始——会话要么接受 `agent: agent.id`，要么接受带类型的哈希形式 `agent: {type: "agent", id: agent.id, version: agent.version}`。

### Minimal / 最小示例

```ruby
# 1. Create the agent (reusable, versioned)
agent = client.beta.agents.create(
  name: "Coding Assistant",
  model: :"claude-opus-5-5",
  system_: "You are a helpful coding assistant.",
  tools: [{type: "agent_toolset_20260401"}]
)

# 2. Start a session
session = client.beta.sessions.create(
  agent: {type: "agent", id: agent.id, version: agent.version},
  environment_id: environment.id,
  title: "Quickstart session"
)
puts "Session ID: #{session.id}"
puts "Trace: https://platform.claude.com/workspaces/default/sessions/#{session.id}"  # swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### Updating an Agent / 更新智能体

Updates create new versions; the agent object is immutable per version.

更新会创建新版本；智能体对象在每个版本内不可变。

```ruby
updated_agent = client.beta.agents.update(
  agent.id,
  version: agent.version,
  system_: "You are a helpful coding agent. Always write tests."
)
puts "New version: #{updated_agent.version}"

# List all versions
client.beta.agents.versions.list(agent.id).auto_paging_each do |version|
  puts "Version #{version.version}: #{version.updated_at.iso8601}"
end

# Archive the agent
archived = client.beta.agents.archive(agent.id)
puts "Archived at: #{archived.archived_at.iso8601}"
```

---

## Send a User Message / 发送用户消息

```ruby
client.beta.sessions.events.send_(
  session.id,
  events: [{
    type: "user.message",
    content: [{type: "text", text: "Review the auth module"}]
  }]
)
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

> 技巧：**流优先：**在发送消息*之前*（或同时）打开流。流只会送达它打开之后发生的事件——先发后开流意味着早期事件会以一批缓冲的形式到达。见 [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns)。

---

## Stream Events (SSE) / 流式事件（SSE）

```ruby
# Open the stream first, then send the user message
stream = client.beta.sessions.events.stream_events(session.id)

client.beta.sessions.events.send_(
  session.id,
  events: [{
    type: "user.message",
    content: [{type: "text", text: "Summarize the repo README"}]
  }]
)

stream.each do |event|
  case event.type
  in :"agent.message"
    event.content.each { |block| print block.text }
  in :"agent.tool_use"
    puts "\n[Using tool: #{event.name}]"
  in :"session.status_idle"
    break
  in :"session.error"
    puts "\n[Error: #{event.error&.message || "unknown"}]"
    break
  else
    # ignore other event types
  end
end
```

> Note: Event `.type` is a Symbol (compare with `:"agent.message"`, not `"agent.message"`).

> 注意：事件的 `.type` 是 Symbol（用 `:"agent.message"` 比较，而不是 `"agent.message"`）。

### Reconnecting and Tailing / 重连与跟随

When reconnecting mid-session, list past events first to dedupe, then tail live events:

会话中途重连时，先列出过往事件以去重，再跟随实时事件：

```ruby
require "set"

stream = client.beta.sessions.events.stream_events(session.id)

# Stream is open and buffering. List history before tailing live.
seen_event_ids = Set.new
client.beta.sessions.events.list(session.id).auto_paging_each { |past| seen_event_ids << past.id }

# Tail live events, skipping anything already seen
stream.each do |event|
  next if seen_event_ids.include?(event.id)
  seen_event_ids << event.id
  case event.type
  in :"agent.message"
    event.content.each { |block| print block.text }
  in :"session.status_idle"
    break
  else
    # ignore other event types
  end
end
```

---

## Provide Custom Tool Result / 提供自定义工具结果

> Note: The Ruby managed-agents bindings for `user.custom_tool_result` are not yet documented in this skill or in the apps source examples. Refer to `shared/managed-agents-events.md` for the wire format and the `anthropic` Ruby gem repository for the corresponding params.

> 注意：`user.custom_tool_result` 的 Ruby managed-agents 绑定尚未在本技能或 apps 源码示例中记录。线上格式见 `shared/managed-agents-events.md`，对应参数见 `anthropic` Ruby gem 仓库。

---

## Poll Events / 轮询事件

```ruby
client.beta.sessions.events.list(session.id).auto_paging_each do |event|
  puts "#{event.type}: #{event.id}"
end
```

---

## Upload a File / 上传文件

```ruby
require "pathname"

file = client.beta.files.upload(file: Pathname("data.csv"))
puts "File ID: #{file.id}"

# Mount in a session
session = client.beta.sessions.create(
  agent: agent.id,
  environment_id: environment.id,
  resources: [
    {
      type: "file",
      file_id: file.id,
      mount_path: "/workspace/data.csv"
    }
  ]
)
```

### Add and Manage Resources on an Existing Session / 在既有会话上添加并管理资源

```ruby
# Attach an additional file to an open session
resource = client.beta.sessions.resources.add(
  session.id,
  type: "file",
  file_id: file.id
)
puts resource.id # "sesrsc_01ABC..."

# List resources on the session
listed = client.beta.sessions.resources.list(session.id)
listed.data.each { |entry| puts "#{entry.id} #{entry.type}" }

# Detach a resource
client.beta.sessions.resources.delete(resource.id, session_id: session.id)
```

---

## List and Download Session Files / 列出并下载会话文件

```ruby
files = client.beta.files.list(scope_id: "sesn_abc123", betas: ["managed-agents-2026-04-01"])
content = client.beta.files.download(files.data[0].id)
File.binwrite("output.txt", content.read)
```

---

## Session Management / 会话管理

```ruby
# List environments
environments = client.beta.environments.list

# Retrieve a specific environment
env = client.beta.environments.retrieve(environment.id)

# Archive an environment (read-only, existing sessions continue)
client.beta.environments.archive(environment.id)

# Delete an environment (only if no sessions reference it)
client.beta.environments.delete(environment.id)

# Delete a session
client.beta.sessions.delete(session.id)
```

---

## MCP Server Integration / MCP 服务器集成

```ruby
# Agent declares MCP server (no auth here - auth goes in a vault)
agent = client.beta.agents.create(
  name: "GitHub Assistant",
  model: :"claude-opus-5-5",
  mcp_servers: [
    {
      type: "url",
      name: "github",
      url: "https://api.githubcopilot.com/mcp/"
    }
  ],
  tools: [
    {type: "agent_toolset_20260401"},
    {type: "mcp_toolset", mcp_server_name: "github"}
  ]
)

# Session attaches vault(s) containing credentials for those MCP server URLs
session = client.beta.sessions.create(
  agent: {type: "agent", id: agent.id, version: agent.version},
  environment_id: environment.id,
  vault_ids: [vault.id]
)
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

创建保险库与添加凭据见 `shared/managed-agents-tools.md` 的"Vaults"一节。

---

## Vaults / 保险库

```ruby
# Create a vault
vault = client.beta.vaults.create(
  display_name: "Alice",
  metadata: {external_user_id: "usr_abc123"}
)
puts vault.id # "vlt_01ABC..."

# Add an OAuth credential
credential = client.beta.vaults.credentials.create(
  vault.id,
  display_name: "Alice's Slack",
  auth: {
    type: "mcp_oauth",
    mcp_server_url: "https://mcp.slack.com/mcp",
    access_token: "xoxp-...",
    expires_at: "2026-04-15T00:00:00Z",
    refresh: {
      token_endpoint: "https://slack.com/api/oauth.v2.access",
      client_id: "1234567890.0987654321",
      scope: "channels:read chat:write",
      refresh_token: "xoxe-1-...",
      token_endpoint_auth: {
        type: "client_secret_post",
        client_secret: "abc123..."
      }
    }
  }
)

# Rotate the credential (e.g., after a token refresh)
client.beta.vaults.credentials.update(
  credential.id,
  vault_id: vault.id,
  auth: {
    type: "mcp_oauth",
    access_token: "xoxp-new-...",
    expires_at: "2026-05-15T00:00:00Z",
    refresh: {refresh_token: "xoxe-1-new-..."}
  }
)

# Archive a vault
client.beta.vaults.archive(vault.id)
```

---

## GitHub Repository Integration / GitHub 仓库集成

Mount a GitHub repository as a session resource (a vault holds the GitHub MCP credential):

把 GitHub 仓库挂载为会话资源（保险库存放 GitHub MCP 凭据）：

```ruby
session = client.beta.sessions.create(
  agent: agent.id,
  environment_id: environment.id,
  vault_ids: [vault.id],
  resources: [
    {
      type: "github_repository",
      url: "https://github.com/org/repo",
      mount_path: "/workspace/repo",
      authorization_token: "ghp_your_github_token"
    }
  ]
)
```

Multiple repositories on the same session:

同一会话上挂载多个仓库：

```ruby
resources = [
  {
    type: "github_repository",
    url: "https://github.com/org/frontend",
    mount_path: "/workspace/frontend",
    authorization_token: "ghp_your_github_token"
  },
  {
    type: "github_repository",
    url: "https://github.com/org/backend",
    mount_path: "/workspace/backend",
    authorization_token: "ghp_your_github_token"
  }
]
```

Rotating a repository's authorization token:

轮换仓库的授权令牌：

```ruby
listed = client.beta.sessions.resources.list(session.id)
repo_resource_id = listed.data.first.id

client.beta.sessions.resources.update(
  repo_resource_id,
  session_id: session.id,
  authorization_token: "ghp_your_new_github_token"
)
```
