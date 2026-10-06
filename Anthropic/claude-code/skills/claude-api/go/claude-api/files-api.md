<!-- BILINGUAL-EN-ZH -->
# Files API - Go / Files API - Go

## Files API / Files API

> **Out of beta.** In current SDKs `client.Beta.Files` has breaking shape changes from previous versions, matching the stable `client.Files` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

> **已正式发布（脱离 beta）。** 在当前 SDK 中，`client.Beta.Files` 的类型结构与先前版本相比存在破坏性变更，与稳定的 `client.Files` 保持一致——请按照 `shared/live-sources.md` 中 Files API 一行的说明进行迁移。下文示例编写于该变更之前。

Under `client.Beta.Files`. Method is **`Upload`** (NOT `New`/`Create`), params struct is `BetaFileUploadParams`. The `File` field takes an `io.Reader`; use `anthropic.File()` to attach a filename + content-type for the multipart encoding.

文件操作位于 `client.Beta.Files` 之下。方法名为 **`Upload`**（而非 `New`/`Create`），参数结构体为 `BetaFileUploadParams`。`File` 字段接受一个 `io.Reader`；请使用 `anthropic.File()` 为 multipart 编码附加文件名和内容类型。

```go
f, _ := os.Open("./upload_me.txt")
defer f.Close()

meta, err := client.Beta.Files.Upload(ctx, anthropic.BetaFileUploadParams{
    File:  anthropic.File(f, "upload_me.txt", "text/plain"),
    Betas: []anthropic.AnthropicBeta{anthropic.AnthropicBetaFilesAPI2025_04_14},
})
// meta.ID is the file_id to reference in subsequent message requests
```

Other `Beta.Files` methods: `List`, `Delete`, `Download`, `GetMetadata`.

其他 `Beta.Files` 方法：`List`、`Delete`、`Download`、`GetMetadata`。

---

