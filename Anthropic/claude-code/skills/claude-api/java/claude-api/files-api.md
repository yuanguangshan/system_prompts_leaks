<!-- BILINGUAL-EN-ZH -->
# Files API - Java / Files API - Java

## Files API / Files API

> **Out of beta.** In current SDKs `client.beta().files()` has breaking shape changes from previous versions, matching the stable `client.files()` - migrate per the Files API row in `shared/live-sources.md`. Examples below predate this.

> **已正式发布（脱离 beta）。** 在当前 SDK 中，`client.beta().files()` 的类型结构与先前版本相比存在破坏性变更，与稳定的 `client.files()` 保持一致——请按照 `shared/live-sources.md` 中 Files API 一行的说明进行迁移。下文示例编写于该变更之前。

Under `client.beta().files()`. File references in messages need the beta message types (non-beta `DocumentBlockParam.Source` has no file-ID variant).

文件操作位于 `client.beta().files()` 之下。在消息中引用文件需要使用 beta 版消息类型（非 beta 版的 `DocumentBlockParam.Source` 没有按文件 ID 引用的变体）。

```java
import com.anthropic.models.beta.files.FileUploadParams;
import com.anthropic.models.beta.files.FileMetadata;
import com.anthropic.models.beta.messages.BetaRequestDocumentBlock;
import com.anthropic.models.beta.messages.BetaFileDocumentSource;
import java.nio.file.Paths;

FileMetadata meta = client.beta().files().upload(
    FileUploadParams.builder()
        .file(Paths.get("/path/to/doc.pdf"))  // or .file(InputStream) or .file(byte[])
        .build());

// Reference in a beta message:
BetaRequestDocumentBlock doc = BetaRequestDocumentBlock.builder()
    .source(BetaFileDocumentSource.builder().fileId(meta.id()).build())
    .build();
```

Other methods: `.list()`, `.delete(String fileId)`, `.download(String fileId)`, `.retrieveMetadata(String fileId)`.

其他方法：`.list()`、`.delete(String fileId)`、`.download(String fileId)`、`.retrieveMetadata(String fileId)`。
