<!-- BILINGUAL-EN-ZH -->
# Files API - TypeScript / Files API - TypeScript

The Files API uploads files for use in Messages API requests. Reference files via `file_id` in content blocks, avoiding re-uploads across multiple API calls.

Files API 用于上传文件以供 Messages API 请求使用。在内容块中通过 `file_id` 引用文件，避免在多次 API 调用间重复上传。

The Files API is out of beta. In current SDKs `client.beta.files` has breaking shape changes from previous versions, matching the stable `client.files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

Files API 已结束测试期。在当前 SDK 中，`client.beta.files` 的结构相对旧版本有破坏性变更，已与稳定的 `client.files` 保持一致——请按照 `shared/live-sources.md` 中 Files API 一行的说明进行迁移。下文示例写于该变更之前。

## Key Facts / 关键事实

- Maximum file size: 500 MB
  - 单个文件大小上限：500 MB
- Total storage: 100 GB per organization
  - 总存储量：每个组织 100 GB
- Files persist until deleted
  - 文件在删除前持续保存
- File operations (upload, list, delete) are free; content used in messages is billed as input tokens
  - 文件操作（上传、列出、删除）免费；在消息中使用的文件内容按输入 token 计费
- Not available on Amazon Bedrock or Google Vertex AI
  - 在 Amazon Bedrock 或 Google Vertex AI 上不可用

---

## Upload a File / 上传文件

```typescript
import Anthropic, { toFile } from "@anthropic-ai/sdk";
import fs from "fs";

const client = new Anthropic();

const uploaded = await client.beta.files.upload({
  file: await toFile(fs.createReadStream("report.pdf"), undefined, {
    type: "application/pdf",
  }),
  betas: ["files-api-2025-04-14"],
});

console.log(`File ID: ${uploaded.id}`);
console.log(`Size: ${uploaded.size_bytes} bytes`);
```

---

## Use a File in Messages / 在消息中使用文件

### PDF / Text Document / PDF / 文本文档

```typescript
const response = await client.beta.messages.create({
  model: "claude-opus-5-5",
  max_tokens: 16000,
  messages: [
    {
      role: "user",
      content: [
        { type: "text", text: "Summarize the key findings in this report." },
        {
          type: "document",
          source: { type: "file", file_id: uploaded.id },
          title: "Q4 Report",
          citations: { enabled: true },
        },
      ],
    },
  ],
  betas: ["files-api-2025-04-14"],
});

console.log(response.content[0].text);
```

---

## Manage Files / 管理文件

### List Files / 列出文件

```typescript
const files = await client.beta.files.list({
  betas: ["files-api-2025-04-14"],
});
for (const f of files.data) {
  console.log(`${f.id}: ${f.filename} (${f.size_bytes} bytes)`);
}
```

### Delete a File / 删除文件

```typescript
await client.beta.files.delete("file_011CNha8iCJcU1wXNR6q4V8w", {
  betas: ["files-api-2025-04-14"],
});
```

### Download a File / 下载文件

```typescript
const response = await client.beta.files.download(
  "file_011CNha8iCJcU1wXNR6q4V8w",
  { betas: ["files-api-2025-04-14"] },
);
const content = Buffer.from(await response.arrayBuffer());
await fs.promises.writeFile("output.txt", content);
```
