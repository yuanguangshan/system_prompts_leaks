<!-- BILINGUAL-EN-ZH -->

# TypeScript Web Artifact Runtime / TypeScript Web 制品运行时

Source for the `@hatch/space-sdk` package consumed by every TypeScript web artifact.

`@hatch/space-sdk` 包的源码，供每个 TypeScript Web 制品使用。

Current TypeScript web artifacts are built by the web artifact builder subagent through the
Rust `web_artifacts.build` tool, which calls `hatch_spaces::build_pipeline::build_space`.
The canonical source root is `workspace/ts-spaces/<slug>`.

当前的 TypeScript Web 制品由 Web 制品构建子代理通过 Rust 的 `web_artifacts.build` 工具构建，该工具调用 `hatch_spaces::build_pipeline::build_space`。规范的源码根目录是 `workspace/ts-spaces/<slug>`。

## Layout / 布局

```
ts-runtime/
├── build.mjs           # produces the local Bun runtime artifacts
├── cloudflare/         # explicit Worker build/typecheck tooling
│   └── tsconfig.worker.base.json
├── sdk/                # @hatch/space-sdk package source
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── index.ts    # server-side surface (defineAction, Ctx, ActionsModule, …)
│       └── client.ts   # browser-side surface (createActionClient)
└── dist/               # gitignored; populated by build.mjs
    └── space-sdk.tgz   # vendored into each TS web artifact via `file:` dep
```

## Building / 构建

Local Bun runtime artifacts:

本地 Bun 运行时产物：

```bash
cd skills/spaces/ts-runtime
bun build.mjs
```

Output lands at `dist/space-sdk.tgz`. Bundle build wires this into the deployed
bundle so `/opt/hatch/skills/spaces/ts-runtime/dist/space-sdk.tgz` exists at
runtime; each scaffolded space's `package.json` references it via  
`"@hatch/space-sdk": "file:/opt/hatch/skills/spaces/ts-runtime/dist/space-sdk.tgz"`.

输出落在 `dist/space-sdk.tgz`。Bundle 构建会把它接入已部署的 bundle，使 `/opt/hatch/skills/spaces/ts-runtime/dist/space-sdk.tgz` 在运行时存在；每个脚手架生成的 space 的 `package.json` 通过  
`"@hatch/space-sdk": "file:/opt/hatch/skills/spaces/ts-runtime/dist/space-sdk.tgz"` 引用它。

## Cloudflare Worker Export / Cloudflare Worker 导出

Cloudflare export is intentionally explicit and is not run by normal
`web_artifacts.build` yet:

Cloudflare 导出被有意设计为显式操作，目前还不会由常规的 `web_artifacts.build` 运行：

```bash
cd skills/spaces/ts-runtime
bun cloudflare/build-cloudflare.mjs \
  --space-dir "$JARVIS_HOME/workspace/ts-spaces/<slug>" \
  --out-dir /tmp/<slug>-cloudflare
```

The exporter writes:

导出器写出：

- `worker.js`: bundled Worker action dispatcher.
  `worker.js`：打包后的 Worker 动作分发器。
- `deploy-manifest.json`: Worker source, client files from `client/dist`, and
  SQL migrations from `drizzle/*.sql`.
  `deploy-manifest.json`：Worker 源码、来自 `client/dist` 的客户端文件，以及来自 `drizzle/*.sql` 的 SQL 迁移。

### Initial Cloudflare Database Seed / Cloudflare 初始数据库种子

The Cloudflare deploy manifest may include an `initialDbSnapshot` field when a
local VM web artifact is shared to Cloudflare for the first time:

当本地虚拟机上的 Web 制品第一次共享到 Cloudflare 时，部署清单可能包含 `initialDbSnapshot` 字段：

```json
{
  "runtime": "hatch-ts-cloudflare-v1",
  "slug": "my-space",
  "name": "My Web Artifact",
  "workerJs": "...",
  "migrations": [],
  "clientFiles": [],
  "initialDbSnapshot": {
    "runtime": "hatch-local-sqlite-snapshot-v1",
    "shortcode": "abc123",
    "slug": "my-space",
    "exportedAtMs": 1725000000000,
    "schemaWatermark": null,
    "payload": {
      "kind": "sqlite_sql",
      "sql": "<sqlite dump sql>"
    }
  }
}
```

`initialDbSnapshot` seeds D1 from a fenced local SQLite snapshot on first
publication. `initialBlobSnapshot` transfers the corresponding objects to R2.
The control plane validates their runtime, shortcode, and slug before upload;
it never overwrites an active database or bucket with a later local seed.
While shared, edits read remote snapshots and deploys apply migrations to D1.
Unshare drains writers and restores both D1 and R2 locally before enabling
local actions. See the [shared-state contract](../../../../docs/spaces-shared-state.md)
for admission, failure, and recovery semantics.

`initialDbSnapshot` 在首次发布时用一个受保护的本地 SQLite 快照为 D1 播种。`initialBlobSnapshot` 把对应的对象传输到 R2。控制平面在上传之前校验它们的 runtime、shortcode 和 slug；它绝不会用更晚的本地种子覆盖活跃的数据库或存储桶。共享期间，编辑读取远端快照，部署向 D1 应用迁移。取消共享时，先排空写入者，并在启用本地操作之前把 D1 和 R2 都恢复到本地。准入、失败与恢复语义见[共享状态契约](../../../../docs/spaces-shared-state.md)。

The build generates a Worker tsconfig and typechecks the space's
`server/src/actions.ts` against a Cloudflare-only `@hatch/space-sdk` shim. That
shim exposes the portable `ctx.db` surface backed by D1 and omits local-only
APIs such as `ctx.inference`, `ctx.agent`, and `ctx.emit`, so unsupported action
code fails through TypeScript rather than source scanning. `SpaceDb` deliberately
does not expose `transaction`.

构建会生成一个 Worker tsconfig，并针对仅面向 Cloudflare 的 `@hatch/space-sdk` 垫片（shim）对 space 的 `server/src/actions.ts` 做类型检查。该垫片暴露由 D1 支撑的可移植 `ctx.db` 接口，并省略仅本地的 API（如 `ctx.inference`、`ctx.agent` 和 `ctx.emit`），因此不受支持的动作代码会经由 TypeScript 报错，而不是靠源码扫描。`SpaceDb` 刻意不暴露 `transaction`。

【评论】通过类型检查层面的垫片让不兼容代码在编译期失败，比运行时拦截或源码扫描更可靠，属于"让非法用法无法通过编译"的工程手法。

## Blob Storage / 二进制对象存储

Local TypeScript web artifact actions can use `ctx.blobs` for binary or opaque object
data that does not belong in `ctx.db`: generated images, thumbnails,
exports, attachments, cached API files, snapshots, and large payloads. Use
`ctx.db` for structured/queryable state, and store blob keys or searchable
metadata there when the UI needs to query across blob-backed records.

本地 TypeScript Web 制品的动作可以用 `ctx.blobs` 存放不属于 `ctx.db` 的二进制或不透明对象数据：生成的图像、缩略图、导出文件、附件、缓存的 API 文件、快照和大载荷。结构化/可查询状态使用 `ctx.db`；当 UI 需要跨基于 blob 的记录做查询时，把 blob 键或可搜索的元数据存到 `ctx.db` 里。

The local VM runtime initializes blob storage lazily on first use:

本地虚拟机运行时在首次使用时才惰性初始化 blob 存储：

```text
<spaceDir>/blobs/objects/<id[0:2]>/<id>   id = sha256(key); sharded
<spaceDir>/blobs/index.sqlite
```

`index.sqlite` tracks `key`, `content_type`, `size_bytes`, `etag`,
`visibility`, `created_at_ms`, `updated_at_ms`, and `object_id`. `etag` is
currently a SHA-256 fingerprint of the stored bytes. Object bytes live under a
hashed, sharded `object_id` path rather than `objects/<key>`, so keys may be
long or contain reserved characters without hitting filesystem name limits;
keys are never used as filesystem paths. Blobs written before this change have
a null `object_id` and are read from the legacy `objects/<key>` path (the
served URL is an opaque base64url token regardless).

`index.sqlite` 跟踪 `key`、`content_type`、`size_bytes`、`etag`、`visibility`、`created_at_ms`、`updated_at_ms` 和 `object_id`。`etag` 目前是所存储字节的 SHA-256 指纹。对象字节存放在经过哈希、分片的 `object_id` 路径下，而不是 `objects/<key>`，因此键可以很长或包含保留字符而不会触到文件系统名称限制；键绝不会被用作文件系统路径。在此变更之前写入的 blob 的 `object_id` 为空，会从旧版 `objects/<key>` 路径读取（无论哪种方式，提供的 URL 都是不透明的 base64url 令牌）。

Example:

示例：

```ts
await ctx.blobs.put("images/avatar.png", bytes, {
  contentType: "image/png",
});
const meta = await ctx.blobs.head("images/avatar.png");
const url = await ctx.blobs.getUrl("images/avatar.png", {
  expiresInSeconds: 600,
});
```

`getUrl()` asks the runtime to mint a fetchable blob URL. Local Muse VM web artifacts
return a document-relative `./blobs/<key>` URL served by the daemon under nginx
bearer auth — relative so it resolves against the `/spaces/v2/<slug>/` document
base whether that sits at the origin root (prod) or behind a `/backend/<sid>/`
reverse-proxy prefix (annotation rig). Cloudflare web artifacts return a Worker-relative
`./blobs/public/<key>` or `./blobs/private/<key>?token=...` URL backed by the
per-web-artifact R2 bucket bound as `BUCKET`, with HMAC-signed tokens for private
blobs. Stateful downloads require a verified viewer and use `Cache-Control:
no-store`, including blobs marked public within the app. Both Cloudflare and VM
blob responses carry CSP `sandbox` and `nosniff`; attachments can display passive
content but cannot execute scripts with the app's origin or permissions.

`getUrl()` 让运行时铸造一个可获取的 blob URL。本地 Muse 虚拟机 Web 制品返回文档相对的 `./blobs/<key>` URL，由守护进程在 nginx bearer 认证下提供——使用相对路径是为了无论它位于源站根目录（生产环境）还是 `/backend/<sid>/` 反向代理前缀之后（标注环境），都能相对 `/spaces/v2/<slug>/` 文档基址解析。Cloudflare Web 制品返回 Worker 相对的 `./blobs/public/<key>` 或 `./blobs/private/<key>?token=...` URL，背后是绑定 `BUCKET` 的每个制品专属的 R2 存储桶，私有 blob 使用 HMAC 签名的令牌。有状态下载需要经过验证的查看者，并使用 `Cache-Control: no-store`，包括在应用内被标记为公开的 blob。Cloudflare 和虚拟机的 blob 响应都带有 CSP `sandbox` 和 `nosniff`；附件可以展示被动内容，但不能以应用的源或权限执行脚本。

Run the local Cloudflare exporter tests with:

运行本地 Cloudflare 导出器测试：

```bash
bun test --timeout 30000 cloudflare/build-cloudflare.test.mjs cloudflare/shared-state.test.ts
```
