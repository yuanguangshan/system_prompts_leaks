<!-- BILINGUAL-EN-ZH -->
# Message Batches API - TypeScript / Message Batches API - TypeScript

The Batches API (`POST /v1/messages/batches`) processes Messages API requests asynchronously at 50% of standard prices.

Batches API（`POST /v1/messages/batches`）以标准价格 50% 的成本异步处理 Messages API 请求。

## Key Facts / 关键事实

- Up to 100,000 requests or 256 MB per batch
  - 每个批次最多 100,000 个请求或 256 MB
- Most batches complete within 1 hour; maximum 24 hours
  - 大多数批次在 1 小时内完成；最长 24 小时
- Results available for 29 days after creation
  - 结果在创建后 29 天内可获取
- 50% cost reduction on all token usage
  - 所有 token 用量成本降低 50%
- All Messages API features supported (vision, tools, caching, etc.)
  - 支持 Messages API 的全部功能（视觉、工具、缓存等）

---

## Create a Batch / 创建批次

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const messageBatch = await client.messages.batches.create({
  requests: [
    {
      custom_id: "request-1",
      params: {
        model: "claude-opus-5-5",
        max_tokens: 16000,
        messages: [
          { role: "user", content: "Summarize climate change impacts" },
        ],
      },
    },
    {
      custom_id: "request-2",
      params: {
        model: "claude-opus-5-5",
        max_tokens: 16000,
        messages: [
          { role: "user", content: "Explain quantum computing basics" },
        ],
      },
    },
  ],
});

console.log(`Batch ID: ${messageBatch.id}`);
console.log(`Status: ${messageBatch.processing_status}`);
```

---

## Poll for Completion / 轮询完成状态

```typescript
let batch;
while (true) {
  batch = await client.messages.batches.retrieve(messageBatch.id);
  if (batch.processing_status === "ended") break;
  console.log(
    `Status: ${batch.processing_status}, processing: ${batch.request_counts.processing}`,
  );
  await new Promise((resolve) => setTimeout(resolve, 60_000));
}

console.log("Batch complete!");
console.log(`Succeeded: ${batch.request_counts.succeeded}`);
console.log(`Errored: ${batch.request_counts.errored}`);
```

---

## Retrieve Results / 获取结果

```typescript
for await (const result of await client.messages.batches.results(
  messageBatch.id,
)) {
  switch (result.result.type) {
    case "succeeded":
      console.log(
        `[${result.custom_id}] ${result.result.message.content[0].text.slice(0, 100)}`,
      );
      break;
    case "errored":
      if (result.result.error.type === "invalid_request") {
        console.log(`[${result.custom_id}] Validation error - fix and retry`);
      } else {
        console.log(`[${result.custom_id}] Server error - safe to retry`);
      }
      break;
    case "expired":
      console.log(`[${result.custom_id}] Expired - resubmit`);
      break;
  }
}
```

---

## Cancel a Batch / 取消批次

```typescript
const cancelled = await client.messages.batches.cancel(messageBatch.id);
console.log(`Status: ${cancelled.processing_status}`); // "canceling"
```
