<!-- BILINGUAL-EN-ZH -->
# Files API - C# / Files API - C#

## Files API / Files API

> **Out of beta.** In current SDKs `client.Beta.Files` has breaking shape changes from previous versions, matching the stable `client.Files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

> **已正式发布（脱离 beta）。** 在当前 SDK 中，`client.Beta.Files` 的类型结构与先前版本相比存在破坏性变更，与稳定的 `client.Files` 保持一致——请按照 `shared/live-sources.md` 中 Files API 一行的说明进行迁移。下文示例编写于该变更之前。

Files live under `client.Beta.Files` (namespace `Anthropic.Models.Beta.Files`). `BinaryContent` implicit-converts from `Stream` and `byte[]`.

文件操作位于 `client.Beta.Files` 之下（命名空间为 `Anthropic.Models.Beta.Files`）。`BinaryContent` 可从 `Stream` 和 `byte[]` 隐式转换。

```csharp
using Anthropic.Models.Beta.Files;
using Anthropic.Models.Beta.Messages;

FileMetadata meta = await client.Beta.Files.Upload(
    new FileUploadParams { File = File.OpenRead("doc.pdf") });

// Referencing the uploaded file requires Beta message types:
new BetaRequestDocumentBlock {
    Source = new BetaFileDocumentSource { FileID = meta.ID },
}
```

The non-beta `DocumentBlockParamSource` union has no file-ID variant - file references need `client.Beta.Messages.Create()`.

非 beta 版的 `DocumentBlockParamSource` 联合类型没有按文件 ID 引用的变体——引用文件需要使用 `client.Beta.Messages.Create()`。

---

