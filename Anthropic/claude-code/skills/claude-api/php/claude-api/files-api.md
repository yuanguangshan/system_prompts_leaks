<!-- BILINGUAL-EN-ZH -->
# Files API - PHP / 文件 API - PHP

## Files API / 文件 API

> **Out of beta.** In current SDKs `$client->beta->files` has breaking shape changes from previous versions, matching the stable `$client->files` - migrate per the Files API row in `shared/live-sources.md`. Example below predates this.

> **已结束测试期。** 在当前 SDK 中，`$client->beta->files` 的结构相较旧版本存在不兼容的破坏性变更，与稳定的 `$client->files` 一致 —— 请按照 `shared/live-sources.md` 中 Files API 一行进行迁移。下方示例基于迁移前的旧写法。

```php
$file = $client->beta->files->upload(
    file: fopen('upload_me.txt', 'r'),
    betas: ['files-api-2025-04-14'],
);
// Reference $file->id as a file content block on ->beta->messages->create().
```
