---
name: "image_search"
description: "Search the web by text query for image URLs and source pages for feeds, artifacts, and visual references. Does not identify a supplied image or person."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->

# Image Search / 图片搜索

Use the bundled CLI to find public images by text query:

使用内置 CLI 通过文本查询查找公开图片：

```sh
/opt/hatch/bin/image-search "Golden Gate Bridge at sunset" --max-results 5
```

Optional result language / 可选的结果语言：

```sh
/opt/hatch/bin/image-search "Paris architecture" \
  --max-results 5 \
  --language fr
```

The CLI returns structured results and omits unavailable fields. Interpret the
URLs as follows:

CLI 返回结构化结果，并省略不可用的字段。按如下方式理解这些 URL：

- `media_url` is the public source image locator and the preferred full-image
  URL when present.
  `media_url` 是公开的源图片定位符，在存在时是首选的完整图片 URL。
- `thumbnail_cdn_url` is Meta's renderable CDN preview and the fallback when
  `media_url` is absent. Treat it as a cache URL, not the sole durable copy of
  an artifact.
  `thumbnail_cdn_url` 是 Meta 可渲染的 CDN 预览，在 `media_url` 缺失时作为后备。将其视为缓存 URL，而不是作品的唯一持久副本。
- `media_handle` and `candidate_ref` are internal, non-renderable fields. Do
  not fetch, display, or pass them as image URLs.
  `media_handle` 和 `candidate_ref` 是内部、不可渲染的字段。不要获取、展示它们，也不要把它们当作图片 URL 传递。
- `page_url` is the source page. Retain it for provenance or attribution.
  `page_url` 是来源页面。为溯源或署名保留它。

The service returns only results with `media_url` or `thumbnail_cdn_url`.
Ignore any result that lacks both renderable URL fields.
Choose the result that best matches the request rather than blindly taking the
first one.

服务只返回带有 `media_url` 或 `thumbnail_cdn_url` 的结果。忽略同时缺少这两个可渲染 URL 字段的任何结果。选择最符合请求的结果，而不是盲目取第一个。

This skill returns locators only: it does not download, upload, or turn an
image into a Muse media reference. A Feed may use a suitable remote image URL.
For an Artifact that needs a durable local copy, pass the chosen locator and
source page through the Artifact's supported media-ingestion path; do not treat
the CDN thumbnail as permanent storage.

此技能只返回定位符：它不下载、不上传，也不把图片转换为 Muse 媒体引用。Feed 可以使用合适的远程图片 URL。对于需要持久本地副本的 Artifact，应将选定的定位符和来源页面通过 Artifact 支持的媒体摄取路径传入；不要把 CDN 缩略图当作永久存储。

For an Artifact, choose a download-safe locator before starting any fetch:

对于 Artifact，在开始任何获取之前先选择一个可安全下载的定位符：

1. Inspect all returned results and prefer an HTTPS `media_url` with no URL
   credentials, query string, or fragment. Preserve result order among those
   simple public URLs.
   检查所有返回的结果，优先选择没有 URL 凭据、查询字符串或片段的 HTTPS `media_url`。在这些简单的公开 URL 之间保持结果顺序。
2. Preflight each candidate like a browser, with a cross-origin referer, then
   fetch one candidate at a time with bounded connect and total timeouts
   (substitute real values for the placeholders):
   像浏览器一样对每个候选做预检，带上跨来源 referer，然后以受限的连接与总超时逐个获取候选（用真实值替换占位符）：

   ```sh
   curl -fsSLI -A 'Mozilla/5.0' -H 'Accept: image/png,image/jpeg,image/gif,image/webp,image/*;q=0.5' -H 'Referer: <any-https-origin-that-is-not-the-image-host>' '<image-url>'
   curl -fsSL --connect-timeout 10 --max-time 60 -A 'Mozilla/5.0' -H 'Accept: image/png,image/jpeg,image/gif,image/webp,image/*;q=0.5' -o '<dest-file>' '<image-url>'
   ```

   Accept only a final `2xx` whose content type and file magic are an image,  
   then copy those bytes into Artifact-owned storage. Prefer PNG or JPEG:  
   WEBP is fine except in a Word document, and AVIF except on a web page,  
   which refuses it at share time with no retry that fixes it. Convert a HEIC  
   to JPEG anywhere. Reject hotlink-blocked, expiring, redirecting, 403/404,  
   non-image, watermarked, or unstable URLs. The referer matters: a  
   hotlink-protected host serves a plain fetch but refuses requests that look  
   like they come from someone else's page, so a URL that fails this probe  
   breaks after the Artifact ships even though it loads for you today. The  
   same preflight applies to any external image URL an Artifact uses, however  
   it was found.
   只接受最终状态为 `2xx` 且内容类型和文件魔数均为图片的结果，
   然后把这些字节复制到 Artifact 自有的存储中。优先选择 PNG 或 JPEG：
   WEBP 可以，但在 Word 文档中除外；AVIF 可以，但在网页上除外，
   网页在分享时会拒绝 AVIF 且没有可修复的重试方法。HEIC 在任何场景都转换为 JPEG。
   拒绝防盗链屏蔽、会过期、重定向、403/404、非图片、带水印或不稳定的 URL。
   referer 很重要：启用防盗链保护的主机会正常响应普通获取，但会拒绝看似来自他人页面的请求，
   因此一个未通过此探测的 URL 即使今天能为你加载，也会在 Artifact 发布后失效。
   无论外部图片 URL 以何种方式找到，Artifact 使用它时都要执行同样的预检。

   【评论】此步骤要求模拟浏览器行为（带跨域 referer 预检）以识别防盗链资源，属于对目标网站访问控制的技术性规避说明；同时也说明 CDN 缩略图链接易失效，需要持久化副本。
3. If no simple `media_url` succeeds, consider a query-bearing `media_url` or
   `thumbnail_cdn_url`. Use the returned locator byte-for-byte; never remove or
   rewrite its query string to make it look simpler.
   如果没有简单的 `media_url` 成功，则考虑带查询字符串的 `media_url` 或 `thumbnail_cdn_url`。逐字节使用返回的定位符；绝不要为了让它看起来更简洁而删除或改写其查询字符串。

Search-result pre-approval records the returned locator's exact path and query,
so a plain GET/HEAD that uses it byte-for-byte is normally allowed without
another prompt even when it has a query string. Do not keep retrying or
substitute an unrelated local image when a fetch fails.

搜索结果的预批准会记录返回定位符的确切路径和查询，因此逐字节使用它的普通 GET/HEAD 通常无需再次提示即被允许，即使它带有查询字符串。当获取失败时，不要反复重试，也不要用无关的本地图片替代。

This is text-to-image search, not reverse-image identification. Do not use it
to identify an unknown person from a supplied photograph.

这是文本搜图，而不是反向图片识别。不要用它从提供的照片识别不明的个人。

【评论】末条为用途限制条款：明确禁止以图搜人，这与大多数人脸识别服务的合规边界一致。
