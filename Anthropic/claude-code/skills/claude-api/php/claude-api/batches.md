<!-- BILINGUAL-EN-ZH -->
# Message Batches - PHP / 消息批处理 - PHP

## Message Batches API / 消息批处理 API

```php
$batch = $client->messages->batches->create(requests: [
    ['customId' => 'req-1', 'params' => ['model' => 'claude-opus-5-5', 'maxTokens' => 1024, 'messages' => [...]]],
    ['customId' => 'req-2', 'params' => [...]],
]);
// Poll $client->messages->batches->retrieve($batch->id) until processingStatus === 'ended',
// then iterate $client->messages->batches->results($batch->id).
```

---
