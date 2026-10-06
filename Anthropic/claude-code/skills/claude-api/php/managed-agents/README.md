<!-- BILINGUAL-EN-ZH -->
# Managed Agents - PHP / 托管代理 - PHP

> **Bindings not shown here:** This README covers the most common managed-agents flows for PHP. If you need a class, method, namespace, field, or behavior that isn't shown, WebFetch the PHP SDK repo **or the relevant docs page** from `shared/live-sources.md` rather than guess. Do not extrapolate from cURL shapes or another language's SDK.

> **此处未展示的绑定：** 本 README 覆盖 PHP 最常见的托管代理流程。如果需要未展示的类、方法、命名空间、字段或行为，请按 `shared/live-sources.md` 对 PHP SDK 仓库**或相关文档页**执行 WebFetch，而不要猜测。不要从 cURL 结构或其他语言的 SDK 推断。

> **Agents are persistent - create once, reference by ID.** Store the agent ID returned by `$client->beta->agents->create` and pass it to every subsequent `->sessions->create`; do not call `agents->create` in the request path. **Recommended:** define agents and environments as version-controlled files synced with `ant apply` - see `shared/anthropic-cli.md` (its live-docs URL is in `shared/live-sources.md`). The CLI owns the control plane (create/update); your code owns the data plane (sessions with the stored ID). The examples below show in-code creation for when you must provision programmatically; in production the create call belongs in setup, not in the request path.

> **代理是持久的——只创建一次，用 ID 引用。** 保存 `$client->beta->agents->create` 返回的代理 ID，并在后续每次 `->sessions->create` 中传入；不要在请求路径中调用 `agents->create`。**推荐做法：** 把代理和环境定义为受版本控制的文件，并用 `ant apply` 同步——参见 `shared/anthropic-cli.md`（其实时文档 URL 在 `shared/live-sources.md` 中）。CLI 负责控制面（创建/更新）；你的代码负责数据面（用已存的 ID 创建会话）。下面的示例展示的是必须以编程方式预配时的代码内创建方式；在生产环境中，创建调用应放在设置阶段，而不是请求路径中。

## Installation / 安装

```bash
composer require "anthropic-ai/sdk" "guzzlehttp/guzzle:^7"
```

## Client Initialization / 客户端初始化

```php
use Anthropic\Client;

// Default (uses ANTHROPIC_API_KEY env var)
$client = new Client();

// Explicit API key
$client = new Client(apiKey: 'your-api-key');
```

---

## Create an Environment / 创建环境

```php
$environment = $client->beta->environments->create(
    name: 'my-dev-env',
    config: ['type' => 'cloud', 'networking' => ['type' => 'unrestricted']],
);
echo "Environment ID: {$environment->id}\n"; // env_...
```

---

## Create an Agent (required first step) / 创建代理（必需的第一步）

> Warning: **There is no inline agent config.** `model`/`system`/`tools` live on the agent object, not the session. Always start with `$client->beta->agents->create()` - the session takes either `agent: $agent->id` or the typed `BetaManagedAgentsAgentParams::with(type: 'agent', id: $agent->id, version: $agent->version)`.

> 警告：**不存在内联的代理配置。** `model`/`system`/`tools` 存放在代理对象上，而不是会话上。始终从 `$client->beta->agents->create()` 开始——会话要么接受 `agent: $agent->id`，要么接受类型化的 `BetaManagedAgentsAgentParams::with(type: 'agent', id: $agent->id, version: $agent->version)`。

### Minimal / 最小示例

```php
use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;

// 1. Create the agent (reusable, versioned)
$agent = $client->beta->agents->create(
    name: 'Coding Assistant',
    model: 'claude-opus-5-5',
    system: 'You are a helpful coding assistant.',
    tools: [
        BetaManagedAgentsAgentToolset20260401Params::with(
            type: 'agent_toolset_20260401',
        ),
    ],
);

// 2. Start a session
$session = $client->beta->sessions->create(
    agent: ['type' => 'agent', 'id' => $agent->id, 'version' => $agent->version],
    environmentID: $environment->id,
    title: 'Quickstart session',
);
echo "Session ID: {$session->id}\n";
echo "Trace: https://platform.claude.com/workspaces/default/sessions/{$session->id}\n"; // swap 'default' for your workspace ID if the API key is not in the Default workspace
```

### Updating an Agent / 更新代理

Updates create new versions; the agent object is immutable per version.

更新会创建新版本；代理对象在每个版本内不可变。

```php
$updatedAgent = $client->beta->agents->update(
    $agent->id,
    version: $agent->version,
    system: 'You are a helpful coding agent. Always write tests.',
);
echo "New version: {$updatedAgent->version}\n";

// List all versions
foreach ($client->beta->agents->versions->list($agent->id)->pagingEachItem() as $version) {
    echo "Version {$version->version}: {$version->updatedAt->format(DateTimeInterface::ATOM)}\n";
}

// Archive the agent
$archived = $client->beta->agents->archive($agent->id);
echo "Archived at: {$archived->archivedAt->format(DateTimeInterface::ATOM)}\n";
```

---

## Send a User Message / 发送用户消息

```php
$client->beta->sessions->events->send(
    $session->id,
    events: [
        [
            'type' => 'user.message',
            'content' => [['type' => 'text', 'text' => 'Review the auth module']],
        ],
    ],
);
```

> Tip: **Stream-first:** Open the stream *before* (or concurrently with) sending the message. The stream only delivers events that occur after it opens - stream-after-send means early events arrive buffered in one batch. See [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns).

> 技巧：**流优先：** 在发送消息*之前*（或与之同时）打开流。流只会传递在它打开之后发生的事件——先发送后开流意味着早期事件会以一批缓冲的形式到达。参见 [Steering Patterns](../../shared/managed-agents-events.md#steering-patterns)。

---

## Stream Events (SSE) / 流式接收事件（SSE）

> Note: **Streaming transporter:** PHP's default buffered PSR-18 client never returns for the open-ended session event stream. Use a streaming Guzzle transporter for `streamStream()` calls - other calls keep the default client.

> 注意：**流式 transporter：** PHP 默认的缓冲式 PSR-18 客户端在面对开放式会话事件流时永远不会返回。对 `streamStream()` 调用使用流式 Guzzle transporter——其他调用保持默认客户端。

【评论】这是一条 PHP 生态特有的实现陷阱：缓冲式 HTTP 客户端要等响应体读完才返回，而事件流是无限长的，因此流式读取必须显式换用流式 transporter。

```php
$streamingClient = new GuzzleHttp\Client(['stream' => true]);

// Open the stream first, then send the user message
$stream = $client->beta->sessions->events->streamStream(
    $session->id,
    requestOptions: ['transporter' => $streamingClient],
);
$client->beta->sessions->events->send(
    $session->id,
    events: [
        [
            'type' => 'user.message',
            'content' => [['type' => 'text', 'text' => 'Summarize the repo README']],
        ],
    ],
);

foreach ($stream as $event) {
    match ($event->type) {
        'agent.message' => array_walk(
            $event->content,
            static fn($block) => $block->type === 'text' ? print($block->text) : null,
        ),
        'agent.tool_use' => print("\n[Using tool: {$event->name}]\n"),
        'session.error' => printf("\n[Error: %s]", $event->error?->message ?? 'unknown'),
        default => null,
    };
    if ($event->type === 'session.status_idle' || $event->type === 'session.error') {
        break;
    }
}
$stream->close();
```

### Reconnecting and Tailing / 重连与持续跟踪

When reconnecting mid-session, list past events first to dedupe, then tail live events:

在会话中途重连时，先列出过往事件以去重，再持续跟踪实时事件：

```php
$stream = $client->beta->sessions->events->streamStream(
    $session->id,
    requestOptions: ['transporter' => $streamingClient],
);

// Stream is open and buffering. List history before tailing live.
$seenEventIds = [];
foreach ($client->beta->sessions->events->list($session->id)->pagingEachItem() as $event) {
    $seenEventIds[$event->id] = true;
}

// Tail live events, skipping anything already seen
foreach ($stream as $event) {
    if (isset($seenEventIds[$event->id])) {
        continue;
    }
    $seenEventIds[$event->id] = true;
    match ($event->type) {
        'agent.message' => array_walk(
            $event->content,
            static fn($block) => $block->type === 'text' ? print($block->text) : null,
        ),
        default => null,
    };
    if ($event->type === 'session.status_idle') {
        break;
    }
}
$stream->close();
```

---

## Provide Custom Tool Result / 提供自定义工具结果

> Note: The PHP managed-agents bindings for `user.custom_tool_result` are not yet documented in this skill or in the apps source examples. Refer to `shared/managed-agents-events.md` for the wire format and the `anthropic-ai/sdk` PHP repository for the corresponding params.

> 注意：`user.custom_tool_result` 的 PHP 托管代理绑定尚未在本 skill 或 apps 源码示例中文档化。线上协议格式参见 `shared/managed-agents-events.md`，对应参数参见 `anthropic-ai/sdk` PHP 仓库。

---

## Poll Events / 轮询事件

```php
foreach ($client->beta->sessions->events->list($session->id)->pagingEachItem() as $event) {
    echo "{$event->type}: {$event->id}\n";
}
```

---

## Upload a File / 上传文件

> Note: **PHP file upload:** The PHP SDK's beta managed-agents file upload binding is not shown in the apps source examples; the canonical PHP example uses raw cURL against `POST /v1/files`. If your codebase prefers the SDK, WebFetch the `anthropic-ai/sdk` PHP repository for the latest binding before writing code.

> 注意：**PHP 文件上传：** PHP SDK 的 beta 托管代理文件上传绑定未出现在 apps 源码示例中；权威的 PHP 示例使用原始 cURL 调用 `POST /v1/files`。如果你的代码库偏好使用 SDK，请在编写代码前对 `anthropic-ai/sdk` PHP 仓库执行 WebFetch 以获取最新绑定。

```php
use Anthropic\Beta\Sessions\BetaManagedAgentsFileResourceParams;

// Raw cURL upload (canonical example from the apps source)
$csvPath = 'data.csv';
$ch = curl_init('https://api.anthropic.com/v1/files');
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST => true,
    CURLOPT_HTTPHEADER => [
        'x-api-key: ' . getenv('ANTHROPIC_API_KEY'),
        'anthropic-version: 2023-06-01',
        'anthropic-beta: files-api-2025-04-14',
    ],
    CURLOPT_POSTFIELDS => ['file' => new CURLFile($csvPath, 'text/csv', 'data.csv')],
]);
$file = json_decode(curl_exec($ch));
echo "File ID: {$file->id}\n";

// Mount in a session
$session = $client->beta->sessions->create(
    agent: $agent->id,
    environmentID: $environment->id,
    resources: [
        BetaManagedAgentsFileResourceParams::with(
            type: 'file',
            fileID: $file->id,
            mountPath: '/workspace/data.csv',
        ),
    ],
);
```

### Add and Manage Resources on an Existing Session / 在已有会话上添加和管理资源

```php
// Attach an additional file to an open session
$resource = $client->beta->sessions->resources->add(
    $session->id,
    type: 'file',
    fileID: $file->id,
);
echo "{$resource->id}\n"; // "sesrsc_01ABC..."

// List resources on the session
$listed = $client->beta->sessions->resources->list($session->id);
foreach ($listed->data as $entry) {
    echo "{$entry->id} {$entry->type}\n";
}

// Detach a resource
$client->beta->sessions->resources->delete($resource->id, sessionID: $session->id);
```

---

## List and Download Session Files / 列出并下载会话文件

```php
$files = $client->beta->files->list(
    scopeID: 'sesn_abc123',
    betas: ['managed-agents-2026-04-01'],
);
$content = $client->beta->files->download($files->data[0]->id);
file_put_contents('output.txt', $content);
```

---

## Session Management / 会话管理

```php
// List environments
$environments = $client->beta->environments->list();

// Retrieve a specific environment
$env = $client->beta->environments->retrieve($environment->id);

// Archive an environment (read-only, existing sessions continue)
$client->beta->environments->archive($environment->id);

// Delete an environment (only if no sessions reference it)
$client->beta->environments->delete($environment->id);

// Delete a session
$client->beta->sessions->delete($session->id);
```

---

## MCP Server Integration / MCP 服务器集成

```php
use Anthropic\Beta\Agents\BetaManagedAgentsAgentToolset20260401Params;
use Anthropic\Beta\Agents\BetaManagedAgentsMCPToolsetParams;
use Anthropic\Beta\Agents\BetaManagedAgentsURLMCPServerParams;
use Anthropic\Beta\Sessions\BetaManagedAgentsAgentParams;

// Agent declares MCP server (no auth here - auth goes in a vault)
$agent = $client->beta->agents->create(
    name: 'GitHub Assistant',
    model: 'claude-opus-5-5',
    mcpServers: [
        BetaManagedAgentsURLMCPServerParams::with(
            type: 'url',
            name: 'github',
            url: 'https://api.githubcopilot.com/mcp/',
        ),
    ],
    tools: [
        BetaManagedAgentsAgentToolset20260401Params::with(type: 'agent_toolset_20260401'),
        BetaManagedAgentsMCPToolsetParams::with(
            type: 'mcp_toolset',
            mcpServerName: 'github',
        ),
    ],
);

// Session attaches vault(s) containing credentials for those MCP server URLs
$session = $client->beta->sessions->create(
    agent: BetaManagedAgentsAgentParams::with(
        type: 'agent',
        id: $agent->id,
        version: $agent->version,
    ),
    environmentID: $environment->id,
    vaultIDs: [$vault->id],
);
```

See `shared/managed-agents-tools.md` §Vaults for creating vaults and adding credentials.

创建 vault 和添加凭据参见 `shared/managed-agents-tools.md` §Vaults。

---

## Vaults / Vaults（保险库）

```php
// Create a vault
$vault = $client->beta->vaults->create(
    displayName: 'Alice',
    metadata: ['external_user_id' => 'usr_abc123'],
);
echo $vault->id . "\n"; // "vlt_01ABC..."

// Add an OAuth credential
$credential = $client->beta->vaults->credentials->create(
    vaultID: $vault->id,
    displayName: "Alice's Slack",
    auth: [
        'type' => 'mcp_oauth',
        'mcp_server_url' => 'https://mcp.slack.com/mcp',
        'access_token' => 'xoxp-...',
        'expires_at' => '2026-04-15T00:00:00Z',
        'refresh' => [
            'token_endpoint' => 'https://slack.com/api/oauth.v2.access',
            'client_id' => '1234567890.0987654321',
            'scope' => 'channels:read chat:write',
            'refresh_token' => 'xoxe-1-...',
            'token_endpoint_auth' => [
                'type' => 'client_secret_post',
                'client_secret' => 'abc123...',
            ],
        ],
    ],
);

// Rotate the credential (e.g., after a token refresh)
$client->beta->vaults->credentials->update(
    $credential->id,
    vaultID: $vault->id,
    auth: [
        'type' => 'mcp_oauth',
        'access_token' => 'xoxp-new-...',
        'expires_at' => '2026-05-15T00:00:00Z',
        'refresh' => ['refresh_token' => 'xoxe-1-new-...'],
    ],
);

// Archive a vault
$client->beta->vaults->archive($vault->id);
```

---

## GitHub Repository Integration / GitHub 仓库集成

Mount a GitHub repository as a session resource (a vault holds the GitHub MCP credential):

把 GitHub 仓库挂载为会话资源（GitHub MCP 凭据存放在 vault 中）：

```php
$session = $client->beta->sessions->create(
    agent: $agent->id,
    environmentID: $environment->id,
    vaultIDs: [$vault->id],
    resources: [
        [
            'type' => 'github_repository',
            'url' => 'https://github.com/org/repo',
            'mount_path' => '/workspace/repo',
            'authorization_token' => 'ghp_your_github_token',
        ],
    ],
);
```

Multiple repositories on the same session:

同一会话上的多个仓库：

```php
$resources = [
    [
        'type' => 'github_repository',
        'url' => 'https://github.com/org/frontend',
        'mount_path' => '/workspace/frontend',
        'authorization_token' => 'ghp_your_github_token',
    ],
    [
        'type' => 'github_repository',
        'url' => 'https://github.com/org/backend',
        'mount_path' => '/workspace/backend',
        'authorization_token' => 'ghp_your_github_token',
    ],
];
```

Rotating a repository's authorization token:

轮换仓库的授权令牌：

```php
$listed = $client->beta->sessions->resources->list($session->id);
$repoResourceId = $listed->data[0]->id;

$client->beta->sessions->resources->update(
    $repoResourceId,
    sessionID: $session->id,
    authorizationToken: 'ghp_your_new_github_token',
);
```
