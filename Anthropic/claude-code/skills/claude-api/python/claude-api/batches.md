<!-- BILINGUAL-EN-ZH -->
# Message Batches API - Python / Message Batches API - Python 版

The Batches API (`POST /v1/messages/batches`) processes Messages API requests asynchronously at 50% of standard prices.

Batches API（`POST /v1/messages/batches`）以标准价格 50% 的成本异步处理 Messages API 请求。

## Key Facts / 关键事实

- Up to 100,000 requests or 256 MB per batch
  每个批次最多 100,000 个请求或 256 MB
- Most batches complete within 1 hour; maximum 24 hours
  大多数批次在 1 小时内完成；最长 24 小时
- Results available for 29 days after creation
  结果在创建后 29 天内可获取
- 50% cost reduction on all token usage
  所有 token 用量成本降低 50%
- All Messages API features supported (vision, tools, caching, etc.)
  支持 Messages API 的全部功能（视觉、工具、缓存等）

---

## Create a Batch / 创建批次

```python
import anthropic
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id="request-1",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-5-5",
                max_tokens=16000,
                messages=[{"role": "user", "content": "Summarize climate change impacts"}]
            )
        ),
        Request(
            custom_id="request-2",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-5-5",
                max_tokens=16000,
                messages=[{"role": "user", "content": "Explain quantum computing basics"}]
            )
        ),
    ]
)

print(f"Batch ID: {message_batch.id}")
print(f"Status: {message_batch.processing_status}")
```

---

## Poll for Completion / 轮询直至完成

```python
import time

while True:
    batch = client.messages.batches.retrieve(message_batch.id)
    if batch.processing_status == "ended":
        break
    print(f"Status: {batch.processing_status}, processing: {batch.request_counts.processing}")
    time.sleep(60)

print("Batch complete!")
print(f"Succeeded: {batch.request_counts.succeeded}")
print(f"Errored: {batch.request_counts.errored}")
```

---

## Retrieve Results / 获取结果

> **Note:** Examples below use `match/case` syntax, requiring Python 3.10+. For earlier versions, use `if/elif` chains instead.

> **注意：** 以下示例使用 `match/case` 语法，要求 Python 3.10+。对于更早的版本，请改用 `if/elif` 链。

```python
for result in client.messages.batches.results(message_batch.id):
    match result.result.type:
        case "succeeded":
            msg = result.result.message
            text = next((b.text for b in msg.content if b.type == "text"), "")
            print(f"[{result.custom_id}] {text[:100]}")
        case "errored":
            if result.result.error.type == "invalid_request":
                print(f"[{result.custom_id}] Validation error - fix request and retry")
            else:
                print(f"[{result.custom_id}] Server error - safe to retry")
        case "canceled":
            print(f"[{result.custom_id}] Canceled")
        case "expired":
            print(f"[{result.custom_id}] Expired - resubmit")
```

---

## Cancel a Batch / 取消批次

```python
cancelled = client.messages.batches.cancel(message_batch.id)
print(f"Status: {cancelled.processing_status}")  # "canceling"
```

---

## List Batches (auto-pagination) / 列出批次（自动分页）

Iterating the return value of any `list()` call auto-paginates across all pages - do not index into `.data` if you want the full set:

迭代任何 `list()` 调用的返回值会自动跨所有页面分页 —— 若要获取完整集合，请不要只索引 `.data`：

```python
for batch in client.messages.batches.list(limit=20):
    print(batch.id, batch.processing_status)
```

For manual control, use `first_page.has_next_page()` / `first_page.get_next_page()` / `first_page.next_page_info()`; `first_page.data` holds the current page's items and `first_page.last_id` is the cursor.

若需手动控制，使用 `first_page.has_next_page()` / `first_page.get_next_page()` / `first_page.next_page_info()`；`first_page.data` 保存当前页的条目，`first_page.last_id` 是游标。

---

## Batch with Prompt Caching / 结合提示词缓存的批次

```python
shared_system = [
    {"type": "text", "text": "You are a literary analyst."},
    {
        "type": "text",
        "text": large_document_text,  # Shared across all requests
        "cache_control": {"type": "ephemeral"}
    }
]

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id=f"analysis-{i}",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-5-5",
                max_tokens=16000,
                system=shared_system,
                messages=[{"role": "user", "content": question}]
            )
        )
        for i, question in enumerate(questions)
    ]
)
```

---

## Full End-to-End Example / 完整端到端示例

```python
import anthropic
import time
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

# 1. Prepare requests
items_to_classify = [
    "The product quality is excellent!",
    "Terrible customer service, never again.",
    "It's okay, nothing special.",
]

requests = [
    Request(
        custom_id=f"classify-{i}",
        params=MessageCreateParamsNonStreaming(
            model="claude-haiku-4-5",
            max_tokens=50,
            messages=[{
                "role": "user",
                "content": f"Classify as positive/negative/neutral (one word): {text}"
            }]
        )
    )
    for i, text in enumerate(items_to_classify)
]

# 2. Create batch
batch = client.messages.batches.create(requests=requests)
print(f"Created batch: {batch.id}")

# 3. Wait for completion
while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(10)

# 4. Collect results
results = {}
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        msg = result.result.message
        results[result.custom_id] = next((b.text for b in msg.content if b.type == "text"), "")

for custom_id, classification in sorted(results.items()):
    print(f"{custom_id}: {classification}")
```
