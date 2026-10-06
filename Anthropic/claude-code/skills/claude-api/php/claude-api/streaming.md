<!-- BILINGUAL-EN-ZH -->
# Streaming - PHP / 流式传输 - PHP

## Streaming / 流式传输

> **Requires SDK v0.5.0+.** v0.4.0 and earlier used a single `$params` array; calling with named parameters throws `Unknown named parameter $model`. Upgrade: `composer require "anthropic-ai/sdk:^0.7"`

> **需要 SDK v0.5.0 及以上版本。** v0.4.0 及更早版本使用单个 `$params` 数组；以命名参数方式调用会抛出 `Unknown named parameter $model`。升级命令：`composer require "anthropic-ai/sdk:^0.7"`

```php
use Anthropic\Messages\RawContentBlockDeltaEvent;
use Anthropic\Messages\TextDelta;

$stream = $client->messages->createStream(
    model: 'claude-opus-5-5',
    maxTokens: 64000,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    if ($event instanceof RawContentBlockDeltaEvent && $event->delta instanceof TextDelta) {
        echo $event->delta->text;
    }
}
```

---
