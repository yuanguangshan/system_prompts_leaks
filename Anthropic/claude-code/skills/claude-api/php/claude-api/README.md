<!-- BILINGUAL-EN-ZH -->
# Claude API - PHP / Claude API - PHP

> **Note:** The PHP SDK is the official Anthropic SDK for PHP. A beta tool runner is available via `$client->beta->messages->toolRunner()`. Structured output helpers are supported via `StructuredOutputModel` classes. Agent SDK is not available. Bedrock, Vertex AI, and Foundry clients are supported.

> **注意：** PHP SDK 是 Anthropic 官方的 PHP SDK。可通过 `$client->beta->messages->toolRunner()` 使用测试版工具运行器。结构化输出辅助通过 `StructuredOutputModel` 类支持。Agent SDK 不可用。支持 Bedrock、Vertex AI 和 Foundry 客户端。

## Installation / 安装

```bash
composer require "anthropic-ai/sdk"
```

## Client Initialization / 客户端初始化

```php
use Anthropic\Client;

// Using API key from environment variable
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
```

### Amazon Bedrock / Amazon Bedrock

```php
use Anthropic\Bedrock\MantleClient;

// Messages-API Bedrock endpoint. Reads AWS credentials from env.
$client = new MantleClient(awsRegion: 'us-east-1');
```

Model IDs on Bedrock take an `anthropic.` prefix - e.g. `model: 'anthropic.claude-opus-5-5'`.

Bedrock 上的模型 ID 带有 `anthropic.` 前缀——例如 `model: 'anthropic.claude-opus-5-5'`。

### Google Vertex AI / Google Vertex AI

```php
use Anthropic\Vertex;

// Constructor is private. Parameter is `location`, not `region`.
$client = Vertex\Client::fromEnvironment(
    location: 'us-east5',
    projectId: 'my-project-id',
);
```

### Anthropic Foundry / Anthropic Foundry

```php
use Anthropic\Foundry;

// Constructor is private. baseUrl or resource is required.
$client = Foundry\Client::withCredentials(
    apiKey: getenv('ANTHROPIC_FOUNDRY_API_KEY'),
    baseUrl: 'https://<resource>.services.ai.azure.com/anthropic/v1',
);
```

---

## Basic Message Request / 基本消息请求

```php
$message = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);

// content is an array of polymorphic blocks (TextBlock, ToolUseBlock,
// ThinkingBlock). Accessing ->text on content[0] without checking the block
// type will throw if the first block is not a TextBlock (e.g., when extended
// thinking is enabled and a ThinkingBlock comes first). Always guard:
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
    }
}
```

If you only want the first text block:

如果你只想要第一个文本块：

```php
foreach ($message->content as $block) {
    if ($block->type === 'text') {
        echo $block->text;
        break;
    }
}
```

---

## Extended Thinking / 扩展思考

**Adaptive thinking is the recommended mode for Claude 4.6+ models.** Claude decides dynamically when and how much to think.

**自适应思考是 Claude 4.6+ 模型的推荐模式。** Claude 动态决定何时思考以及思考多少。

```php
use Anthropic\Messages\ThinkingBlock;

$message = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    thinking: ['type' => 'adaptive', 'display' => 'summarized'], // display opt-in: default is omitted (empty thinking text) on Fable 5/5.1, Mythos 5/5.1, Claude Opus 5.5, Claude Opus 5, Opus 4.8/4.7, Claude Sonnet 5.5, and Claude Sonnet 5
    messages: [
        ['role' => 'user', 'content' => 'Solve: 27 * 453'],
    ],
);

// ThinkingBlock(s) precede TextBlock in content
foreach ($message->content as $block) {
    if ($block instanceof ThinkingBlock) {
        echo "Thinking:\n{$block->thinking}\n\n";
        // $block->signature is an opaque string - preserve verbatim if
        // passing thinking blocks back in multi-turn conversations
    } elseif ($block->type === 'text') {
        echo "Answer: {$block->text}\n";
    }
}
```

> **Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6:** Use adaptive thinking (above). `['type' => 'enabled', 'budgetTokens' => N]` is removed on Fable 5, Claude Opus 5.5, Claude Opus 5, Opus 4.8, and 4.7 (400 if sent); deprecated on Opus 4.6 and Sonnet 4.6.  
> **Claude Opus 5.5:** thinking is always on - omit `thinking:` (or send `['type' => 'adaptive']`, which is equivalent); `['type' => 'disabled']` returns a 400 at every effort, as does a thinking budget. Control depth with the `outputConfig:` effort instead - the default is `medium` on this model, where Claude Opus 5 defaults to `high`.  
> **Claude Opus 5:** thinking is on by default - omitting `thinking:` runs adaptive (`['type' => 'adaptive']` is equivalent), unlike Opus 4.8/4.7 where omitting it meant no thinking. `['type' => 'disabled']` is accepted only at effort `high` or lower; pairing it with `xhigh`/`max` returns a 400.  
> **Older models:** Use `thinking: ['type' => 'enabled', 'budgetTokens' => N]` (budget must be < `maxTokens`, min 1024).

> **Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8、Opus 4.7、Opus 4.6 和 Sonnet 4.6：** 使用自适应思考（如上）。`['type' => 'enabled', 'budgetTokens' => N]` 在 Fable 5、Claude Opus 5.5、Claude Opus 5、Opus 4.8 和 4.7 上已被移除（发送则返回 400）；在 Opus 4.6 和 Sonnet 4.6 上已弃用。  
> **Claude Opus 5.5：** 思考始终开启——省略 `thinking:`（或发送等价的 `['type' => 'adaptive']`）；`['type' => 'disabled']` 在任何 effort 级别都返回 400，思考预算同样如此。请改用 `outputConfig:` effort 控制深度——该模型默认为 `medium`，而 Claude Opus 5 默认为 `high`。  
> **Claude Opus 5：** 思考默认开启——省略 `thinking:` 时运行自适应模式（等价于 `['type' => 'adaptive']`），这与 Opus 4.8/4.7 不同，后者省略即表示不思考。`['type' => 'disabled']` 仅在 effort 为 `high` 或更低时被接受；与 `xhigh`/`max` 搭配则返回 400。  
> **更早的模型：** 使用 `thinking: ['type' => 'enabled', 'budgetTokens' => N]`（预算必须小于 `maxTokens`，最小值为 1024）。

`$block->type === 'thinking'` also works for the check; `instanceof` narrows for PHPStan.

`$block->type === 'thinking'` 也可用于该检查；`instanceof` 则可为 PHPStan 收窄类型。

---

## Prompt Caching / 提示词缓存

`system:` takes an array of text blocks; set `cacheControl` on the last block. Array-shape syntax (camelCase keys) is idiomatic. For placement patterns and the silent-invalidator audit checklist, see `shared/prompt-caching.md`.

`system:` 接受文本块数组；在最后一个块上设置 `cacheControl`。数组形态语法（驼峰式键名）是惯用写法。放置模式与静默失效审查清单参见 `shared/prompt-caching.md`。

```php
$message = $client->messages->create(
    model: 'claude-opus-5-5',
    maxTokens: 16000,
    system: [
        ['type' => 'text', 'text' => $longSystemPrompt, 'cacheControl' => ['type' => 'ephemeral']],
    ],
    messages: [['role' => 'user', 'content' => 'Summarize the key points']],
);
```

For 1-hour TTL: `'cacheControl' => ['type' => 'ephemeral', 'ttl' => '1h']`. There's also a top-level `cacheControl:` on `messages->create(...)` that auto-places on the last cacheable block.

如需 1 小时 TTL：`'cacheControl' => ['type' => 'ephemeral', 'ttl' => '1h']`。`messages->create(...)` 上还有一个顶层 `cacheControl:`，它会自动放置在最后一个可缓存的块上。

Verify hits via `$message->usage->cacheCreationInputTokens` / `$message->usage->cacheReadInputTokens`.

通过 `$message->usage->cacheCreationInputTokens` / `$message->usage->cacheReadInputTokens` 验证缓存命中。

---

## Stop Details / 停止详情

When `stopReason` is `'refusal'`, the response includes structured `stopDetails`:

当 `stopReason` 为 `'refusal'`（拒答）时，响应中包含结构化的 `stopDetails`：

```php
if ($message->stopReason === 'refusal' && $message->stopDetails !== null) {
    echo "Category: " . $message->stopDetails->category . "\n";     // e.g. "cyber", "bio", "reasoning_extraction", "frontier_llm", or null - see docs for the full set
    echo "Explanation: " . $message->stopDetails->explanation . "\n";
}
```

**Refusal fallbacks (Claude Fable 5.1) - opt in by default.** Fallbacks are opt-in: without them a refused request simply stops. New `claude-fable-5-1` code should include the server-side `fallbacks` parameter (beta header `server-side-fallback-2026-06-01`, fallback model `claude-opus-4-8`, on the beta messages call) by default. The exact PHP binding (and the client-side middleware for providers without server-side support) is not documented here - WebFetch the PHP SDK repo's `examples/` from `shared/live-sources.md`; full semantics in `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason.

**拒答回退（Claude Fable 5.1）——默认要求显式启用。** 回退是可选择加入的：没有回退时，被拒答的请求会直接停止。新的 `claude-fable-5-1` 代码应默认包含服务端 `fallbacks` 参数（beta 头为 `server-side-fallback-2026-06-01`，回退模型为 `claude-opus-4-8`，作用于 beta 消息调用）。确切的 PHP 绑定（以及为不支持服务端能力的提供商准备的客户端中间件）未在本文档中说明——请按 `shared/live-sources.md` 从 PHP SDK 仓库 WebFetch 其 `examples/`；完整语义见 `shared/model-migration.md` -> Migrating to Claude Fable 5.1 -> `refusal` stop reason。

---

## Error Type / 错误类型

`APIStatusException` exposes a `->type` property for programmatic error classification:

`APIStatusException` 暴露了 `->type` 属性，用于以编程方式对错误进行分类：

```php
try {
    $client->messages->create(...);
} catch (\Anthropic\Core\Exceptions\APIStatusException $e) {
    echo $e->type?->value;  // "rate_limit_error", "overloaded_error", etc.
}
```
