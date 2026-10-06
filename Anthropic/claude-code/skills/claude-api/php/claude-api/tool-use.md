<!-- BILINGUAL-EN-ZH -->
# Tool Use - PHP / 工具使用 - PHP

For conceptual overview (tool definitions, tool choice, tips), see [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md).

概念性概述（工具定义、工具选择、相关技巧）请参见 [shared/tool-use-concepts.md](../../shared/tool-use-concepts.md)。

## Tool Use / 工具使用

### Tool Runner (Beta) / 工具运行器（Beta）

**Beta:** The PHP SDK provides a tool runner via `$client->beta->messages->toolRunner()`. Define tools with `BetaRunnableTool` - a definition array plus a `run` closure:

**Beta：** PHP SDK 通过 `$client->beta->messages->toolRunner()` 提供工具运行器。使用 `BetaRunnableTool` 定义工具——一个定义数组加一个 `run` 闭包：

```php
use Anthropic\Lib\Tools\BetaRunnableTool;

$weatherTool = new BetaRunnableTool(
    definition: [
        'name' => 'get_weather',
        'description' => 'Get the current weather for a location.',
        'inputSchema' => [
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
    run: function (array $input): string {
        return "The weather in {$input['location']} is sunny and 72°F.";
    },
);

$runner = $client->beta->messages->toolRunner(
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'What is the weather in Paris?']],
    model: 'claude-opus-5-5',
    tools: [$weatherTool],
);

foreach ($runner as $message) {
    foreach ($message->content as $block) {
        if ($block->type === 'text') {
            echo $block->text;
        }
    }
}
```

### Manual Loop / 手动循环

Tools are passed as arrays. **The SDK uses camelCase keys** (`inputSchema`, `toolUseID`, `stopReason`) and auto-maps to the API's snake_case on the wire - since v0.5.0. See [shared tool use concepts](../../shared/tool-use-concepts.md) for the loop pattern.

工具以数组形式传入。**SDK 使用 camelCase（小驼峰）键名**（`inputSchema`、`toolUseID`、`stopReason`），并自 v0.5.0 起在线上传输时自动映射为 API 的 snake_case。循环模式请参见[共享的工具使用概念](../../shared/tool-use-concepts.md)。

```php
use Anthropic\Messages\ToolUseBlock;

$tools = [
    [
        'name' => 'get_weather',
        'description' => 'Get the current weather in a given location',
        'inputSchema' => [  // camelCase, not input_schema
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City and state'],
            ],
            'required' => ['location'],
        ],
    ],
];

$messages = [['role' => 'user', 'content' => 'What is the weather in SF?']];

$response = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    tools: $tools,
    messages: $messages,
);

while ($response->stopReason === 'tool_use') {  // camelCase property
    $toolResults = [];
    foreach ($response->content as $block) {
        if ($block instanceof ToolUseBlock) {
            // $block->name  : string               - tool name to dispatch on
            // $block->input : array<string,mixed>  - parsed JSON input
            // $block->id    : string               - pass back as toolUseID
            $result = executeYourTool($block->name, $block->input);
            $toolResults[] = [
                'type' => 'tool_result',
                'toolUseID' => $block->id,  // camelCase, not tool_use_id
                'content' => $result,
            ];
        }
    }

    // Append assistant turn + user turn with tool results
    $messages[] = ['role' => 'assistant', 'content' => $response->content];
    $messages[] = ['role' => 'user', 'content' => $toolResults];

    $response = $client->messages->create(
        model: 'claude-opus-5-5',
        maxTokens: 16000,
        tools: $tools,
        messages: $messages,
    );
}

// Final text response
foreach ($response->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

`$block->type === 'tool_use'` also works; `instanceof ToolUseBlock` narrows for PHPStan.

`$block->type === 'tool_use'` 的写法同样可行；`instanceof ToolUseBlock` 可为 PHPStan 收窄类型。


---

## Structured Outputs / 结构化输出

### Using StructuredOutputModel (Recommended) / 使用 StructuredOutputModel（推荐）

Define a PHP class implementing `StructuredOutputModel` and pass it as `outputConfig`:

定义一个实现 `StructuredOutputModel` 的 PHP 类，并将其作为 `outputConfig` 传入：

```php
use Anthropic\Lib\Contracts\StructuredOutputModel;
use Anthropic\Lib\Concerns\StructuredOutputModelTrait;
use Anthropic\Lib\Attributes\Constrained;

class Person implements StructuredOutputModel
{
    use StructuredOutputModelTrait;

    #[Constrained(description: 'Full name')]
    public string $name;

    public int $age;

    public ?string $email = null;  // nullable = optional field
}

$message = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Generate a profile for Alice, age 30']],
    outputConfig: ['format' => Person::class],
);

$person = $message->parsedOutput();  // Person instance
echo $person->name;
```

Types are inferred from PHP type hints. Use `#[Constrained(description: '...')]` to add descriptions. Nullable properties (`?string`) become optional fields.

类型从 PHP 类型声明自动推断。使用 `#[Constrained(description: '...')]` 可添加字段描述。可空属性（`?string`）会成为可选字段。

### Raw Schema / 原始 Schema

```php
$message = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    messages: [['role' => 'user', 'content' => 'Extract: John (john@co.com), Enterprise plan']],
    outputConfig: [
        'format' => [
            'type' => 'json_schema',
            'schema' => [
                'type' => 'object',
                'properties' => [
                    'name' => ['type' => 'string'],
                    'email' => ['type' => 'string'],
                    'plan' => ['type' => 'string'],
                ],
                'required' => ['name', 'email', 'plan'],
                'additionalProperties' => false,
            ],
        ],
    ],
);

// First text block contains valid JSON
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        $data = json_decode($block->text, true);
        break;
    }
}
```

---

## Beta Features & Anthropic-Defined Tools / Beta 功能与 Anthropic 定义的工具

**`betas:` is NOT a param on `$client->messages->create()`** - it only exists on the beta namespace. Use it for features that need an explicit opt-in header:

**`betas:` 并不是 `$client->messages->create()` 的参数**——它只存在于 beta 命名空间上。需要显式 opt-in 请求头的功能才使用它：

```php
use Anthropic\Beta\Messages\BetaRequestMCPServerURLDefinition;

$response = $client->beta->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    mcpServers: [
        BetaRequestMCPServerURLDefinition::with(
            name: 'my-server',
            url: 'https://example.com/mcp',
        ),
    ],
    betas: ['mcp-client-2025-11-20'],  // only valid on ->beta->messages
    messages: [['role' => 'user', 'content' => 'Use the MCP tools']],
);
```

### Task budgets / 任务预算

```php
$response = $client->beta->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    outputConfig: ['taskBudget' => ['type' => 'tokens', 'total' => 64000]],
    tools: [...],
    messages: [...],
    betas: ['task-budgets-2026-03-13'],
);
```

### Cache diagnostics / 缓存诊断

Pass the previous response's `id` on the next request; print the `diagnostics` object on the response:

在下一次请求中传入上一个响应的 `id`，并打印响应中的 `diagnostics` 对象：

```php
$r2 = $client->beta->messages->create(
    model: 'claude-opus-5-5', maxTokens: 1024,
    diagnostics: ['previousMessageId' => $r1->id],
    betas: ['cache-diagnosis-2026-04-07'],
    messages: [...],
);
```

**Anthropic-defined tools** (bash, web_search, text_editor, code_execution) are GA and work on both paths. Of these, web_search and code_execution are server-executed; bash and text_editor are client-executed (you handle the `tool_use` locally) - `Anthropic\Messages\ToolBash20250124` / `WebSearchTool20260209` / `ToolTextEditor20250728` / `CodeExecutionTool20260120` for non-beta, `Anthropic\Beta\Messages\BetaToolBash20250124` / `BetaWebSearchTool20260209` / `BetaToolTextEditor20250728` / `BetaCodeExecutionTool20260120` for beta. No `betas:` header needed for these.

**Anthropic 定义的工具**（bash、web_search、text_editor、code_execution）已正式发布（GA），在两条路径上均可用。其中 web_search 与 code_execution 由服务端执行；bash 与 text_editor 由客户端执行（由你在本地处理 `tool_use`）——非 beta 路径使用 `Anthropic\Messages\ToolBash20250124` / `WebSearchTool20260209` / `ToolTextEditor20250728` / `CodeExecutionTool20260120`，beta 路径使用 `Anthropic\Beta\Messages\BetaToolBash20250124` / `BetaWebSearchTool20260209` / `BetaToolTextEditor20250728` / `BetaCodeExecutionTool20260120`。这些工具无需 `betas:` 请求头。

### Tool search (non-beta, server-side) / 工具搜索（非 beta，服务端）

```php
tools: [
    ['type' => 'tool_search_tool_regex_20251119', 'name' => 'tool_search_tool_regex'],
    ['name' => 'get_weather', 'description' => '...', 'inputSchema' => [...], 'deferLoading' => true],
    // ... other user tools with 'deferLoading' => true
],
```

### Memory tool (non-beta, client-executed) / 记忆工具（非 beta，客户端执行）

Declare `['type' => 'memory_20250818', 'name' => 'memory']`. Handle the `tool_use` by reading/writing files under a fixed `/memories` directory. **Validate every model-supplied path**: resolve to its canonical form and verify it remains within the memory directory; reject traversal (`..`, symlinks) - see `shared/tool-use-concepts.md` § Client-Side Tools.

声明 `['type' => 'memory_20250818', 'name' => 'memory']`。通过在固定的 `/memories` 目录下读写文件来处理 `tool_use`。**必须校验模型提供的每一个路径**：将其解析为规范形式，并确认其仍位于记忆目录之内；拒绝路径穿越（`..`、符号链接）——参见 `shared/tool-use-concepts.md` § Client-Side Tools。

【评论】“校验模型提供的每一个路径”是典型的客户端工具安全要求：模型输出可能被提示词注入操纵，客户端必须自行完成路径规范化与目录边界检查，不能直接信任 `tool_use` 中的输入。

---
