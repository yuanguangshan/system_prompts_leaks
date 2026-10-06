<!-- BILINGUAL-EN-ZH -->
# Files API - Python / 文件 API - Python

The Files API uploads files for use in Messages API requests. Reference files via `file_id` in content blocks, avoiding re-uploads across multiple API calls.

Files API 用于上传文件以供 Messages API 请求使用。在内容块中通过 `file_id` 引用文件，避免在多次 API 调用中重复上传。

The Files API is out of beta. In current SDKs `client.beta.files` has breaking shape changes from previous versions, matching the stable `client.files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

Files API 已结束测试期。在当前 SDK 中，`client.beta.files` 的结构相对旧版本有破坏性变更，与稳定的 `client.files` 保持一致——请按 `shared/live-sources.md` 中 Files API 一行进行迁移。下方示例编写于此变更之前。

## Key Facts / 关键事实

- Maximum file size: 500 MB
  单个文件大小上限：500 MB
- Total storage: 100 GB per organization
  总存储空间：每个组织 100 GB
- Files persist until deleted
  文件会一直保留，直到被删除
- File operations (upload, list, delete) are free; content used in messages is billed as input tokens
  文件操作（上传、列出、删除）免费；在消息中使用的文件内容按输入 token 计费
- Not available on Amazon Bedrock or Google Vertex AI
  在 Amazon Bedrock 或 Google Vertex AI 上不可用

---

## Upload a File / 上传文件

The `file` argument accepts a `(filename, content, content_type)` tuple, a `pathlib.Path` (or any `PathLike` - read for you, async-safe with `AsyncAnthropic`), or an open binary file object.

`file` 参数接受 `(filename, content, content_type)` 元组、`pathlib.Path`（或任何 `PathLike`——会替你读取，配合 `AsyncAnthropic` 时是异步安全的），或一个已打开的二进制文件对象。

```python
import anthropic
from pathlib import Path

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("report.pdf", open("report.pdf", "rb"), "application/pdf"),
)
# or: client.beta.files.upload(file=Path("report.pdf"))
print(f"File ID: {uploaded.id}")
print(f"Size: {uploaded.size_bytes} bytes")
```

---

## Use a File in Messages / 在消息中使用文件

### PDF / Text Document / PDF / 文本文档

```python
response = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Summarize the key findings in this report."},
            {
                "type": "document",
                "source": {"type": "file", "file_id": uploaded.id},
                "title": "Q4 Report",           # optional
                "citations": {"enabled": True}   # optional, enables citations
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
for block in response.content:
    if block.type == "text":
        print(block.text)
```

### Image / 图像

```python
image_file = client.beta.files.upload(
    file=("photo.png", open("photo.png", "rb"), "image/png"),
)

response = client.beta.messages.create(
    model="claude-opus-5-5",
    max_tokens=16000,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {
                "type": "image",
                "source": {"type": "file", "file_id": image_file.id}
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
```

---

## Manage Files / 管理文件

### List Files / 列出文件

Iterate the list result directly - the SDK auto-paginates across all pages. Only use `.data` if you want the first page only.

直接迭代列表结果即可——SDK 会在所有页面间自动分页。只有当你只想要第一页时才使用 `.data`。

```python
for f in client.beta.files.list():
    print(f"{f.id}: {f.filename} ({f.size_bytes} bytes)")
```

### Get File Metadata / 获取文件元数据

```python
file_info = client.beta.files.retrieve_metadata("file_011CNha8iCJcU1wXNR6q4V8w")
print(f"Filename: {file_info.filename}")
print(f"MIME type: {file_info.mime_type}")
```

### Delete a File / 删除文件

```python
client.beta.files.delete("file_011CNha8iCJcU1wXNR6q4V8w")
```

### Download a File / 下载文件

Only files created by the code execution tool or skills can be downloaded (not user-uploaded files).

只有由代码执行工具或技能创建的文件才能被下载（用户上传的文件不能）。

```python
file_content = client.beta.files.download("file_011CNha8iCJcU1wXNR6q4V8w")
file_content.write_to_file("output.txt")
```

---

## Full End-to-End Example / 完整端到端示例

Upload a document once, ask multiple questions about it:

上传文档一次，围绕它提出多个问题：

```python
import anthropic

client = anthropic.Anthropic()

# 1. Upload once
uploaded = client.beta.files.upload(
    file=("contract.pdf", open("contract.pdf", "rb"), "application/pdf"),
)
print(f"Uploaded: {uploaded.id}")

# 2. Ask multiple questions using the same file_id
questions = [
    "What are the key terms and conditions?",
    "What is the termination clause?",
    "Summarize the payment schedule.",
]

for question in questions:
    response = client.beta.messages.create(
        model="claude-opus-5-5",
        max_tokens=16000,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": question},
                {
                    "type": "document",
                    "source": {"type": "file", "file_id": uploaded.id}
                }
            ]
        }],
        betas=["files-api-2025-04-14"],
    )
    print(f"\nQ: {question}")
    text = next((b.text for b in response.content if b.type == "text"), "")
    print(f"A: {text[:200]}")

# 3. Clean up when done
client.beta.files.delete(uploaded.id)
```
