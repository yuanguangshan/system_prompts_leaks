---
name: library
description: Use ChatGPT Library when the user mentions their Library, asks to find or work with a Library-backed file, Site, or named file that may be in the Library, or wants to organize Library folders, restore a previous version, or share a native Library file or folder. Also use it when the user wants a user-facing file or reusable artifact created or updated, even if they do not mention the Library. It can search and read Library content, bring files into local workflows, save new deliverables, and update existing Library files while preserving their identity and version history.
---
<!-- BILINGUAL-EN-ZH -->

# ChatGPT Library

Use this as the top-level router for persistent ChatGPT Library files. Ground
the target in Library, choose one content-access route, and preserve Library
identity through every local edit and writeback.

将此用作持久化 ChatGPT Library 文件的顶层路由。把目标锚定在 Library 中，选择一条内容访问路径，并在每一次本地编辑和写回中保持 Library 身份。

The Library first-party app, `connector_openai_library`, owns authenticated
Library operations:

Library 第一方应用 `connector_openai_library` 负责经过身份验证的 Library 操作：

- `list`, `search`, `read`, and `find` inspect Library content.
  `list`、`search`、`read` 和 `find` 用于检视 Library 内容。
- `prepare_materialize` makes resolved Library files available to local tools.
  `prepare_materialize` 将已解析的 Library 文件提供给本地工具使用。
- `create_library_file`, `replace_library_file`, and `manage_library` write or organize Library content.
  `create_library_file`、`replace_library_file` 和 `manage_library` 写入或整理 Library 内容。
- `share` grants or revokes access; follow its current app-provided description and schema.
  `share` 授予或撤销访问权限；遵循其当前由应用提供的描述和模式。
- When surfaced, `prepare_uploads` and `finalize_uploads` handle prepared
  uploads.
  在可用时，`prepare_uploads` 和 `finalize_uploads` 处理预备上传。
- App-only file `@` mention search resolves selected Library items.
  仅限应用的文件 `@` 提及搜索解析所选的 Library 条目。

The runtime supplies the schemas and decides which tools are available. This
skill does not expose tools or a local MCP server.

运行时提供各模式并决定哪些工具可用。本技能不暴露工具，也不提供本地 MCP 服务器。

## Workflow / 工作流

1. Ground the target.
   将目标锚定。
   - Decode a selected Library `@` mention and reuse its identifiers.
     - 解码所选的 Library `@` 提及并复用其标识符。
   - Use `search` for a filename, title, description, or content query.
     - 文件名、标题、描述或内容查询使用 `search`。
   - Use `list` for recent files, folders, or inventory.
     - 最近的文件、文件夹或清单使用 `list`。
   - If the user supplied a local path or explicitly said the file is local,
     stay with local tools.
     - 如果用户提供了本地路径或明确说明文件在本地，则继续使用本地工具。
2. Choose one access route.
   选择一条访问路径。
   - Use `read` when Library can supply the content the task needs.
     - 当 Library 能提供任务所需内容时使用 `read`。
   - Use `find` for literal or regex matching inside known files.
     - 在已知文件内做字面或正则匹配使用 `find`。
   - Materialize when editing, scripts, visual inspection, generation, byte
     comparison, or another local tool needs file bytes.
     - 当编辑、脚本、可视化检查、生成、字节比较或其他本地工具需要文件字节时，执行物化（materialize）。
3. Preserve identity when writing.
   写入时保持身份。
   - Create only when no Library identity exists or the user wants a copy.
     - 仅在不存在 Library 身份或用户想要副本时才创建。
   - Replace the same `library_file_id` after editing an existing item.
     - 编辑现有条目后，替换同一个 `library_file_id`。
   - Use `manage_library` for folders, node changes, deletion, and restore.
     - 文件夹、节点变更、删除和恢复使用 `manage_library`。

## Use Current Helpers / 使用当前辅助脚本

1. Before downloading, remove any downloaded copies of helpers from previous
   runs in the scoped workspace folder
   1. 下载前，先删除此前运行下载到作用域工作区文件夹中的辅助脚本副本
2. Fetch the helper and all its companion files from the current Library skill
   source into one new private directory.
   2. 从当前 Library 技能源获取该辅助脚本及其全部配套文件，放入一个新建的私有目录。
3. Reuse the downloaded helpers for every call and retry within this request
   or run.
   3. 在本次请求或运行中，每次调用和重试都复用已下载的辅助脚本。

## Routing Rules / 路由规则

| User need | Required route |
| --- | --- |
| Selected Library `@` mention | Decode its `oai-library://...` URI and use the returned identifiers. Search only if an identifier is missing, metadata must be refreshed, or the target is ambiguous. |
| Exact filename or title | Use `search`; quote the actual name and set `search_title_only=true`. |
| File described by purpose or contents | Use ordinary Library `search`. Do not treat a descriptive phrase as an exact filename. |
| Recent files, folders, or inventory | Use `list`; pass a returned `next_cursor` as `cursor` to continue. |
| Facts, summary, or comparison from named files | Resolve every file, then use `read` on every selected file. Search snippets alone are insufficient. |
| Literal or regex match in known files | Use `find`; follow with `read` only when the match needs more context. |
| Shared writing block (`library_artifact_type: writing_block`) | Use `read` to reconstruct its complete local text; while `has_more`, continue with `next_read`. Preserve line boundaries, verify `size_bytes`, and retain the authoritative `version_id`; do not materialize. |
| Local bytes from a `list` or `search` result | Reuse the complete earlier result when its file identifiers and path remain current; repeat `list` or `search` only if that metadata is missing, stale, or ambiguous. Pass it directly to the bundled stdin-only download helper. Do not call Library `read` or `prepare_materialize` first. |
| Local bytes from a resolved reference that did not come from `list` or `search` | Use `prepare_materialize`; read [materialization.md](references/materialization.md). |
| Explicit local path | Use local tools. Do not send local paths to Library `read` or `find`. |
| New local deliverable | Choose one create route below. |
| Edit a Library-backed file | For `library_artifact_type: site`, use Sites; never materialize, replace, or restore its projection. Otherwise materialize if needed, edit and validate locally, then replace the same `library_file_id`. |
| Create, move, rename, or delete Library nodes | Use `manage_library`; read [library-management.md](references/library-management.md) before mutating. |
| Restore an earlier version | Use `manage_library` with `restore_version`; read [library-management.md](references/library-management.md). |

| 用户需求 | 所需路径 |
| --- | --- |
| 所选 Library `@` 提及 | 解码其 `oai-library://...` URI 并使用返回的标识符。仅当缺少标识符、需要刷新元数据或目标有歧义时才搜索。 |
| 确切的文件名或标题 | 使用 `search`；引用实际名称并设置 `search_title_only=true`。 |
| 按用途或内容描述的文件 | 使用普通 Library `search`。不要把描述性短语当作确切文件名。 |
| 最近的文件、文件夹或清单 | 使用 `list`；将返回的 `next_cursor` 作为 `cursor` 传入以继续。 |
| 来自指定文件的事实、摘要或比较 | 解析每个文件，然后对每个所选文件使用 `read`。仅靠搜索摘要不够。 |
| 已知文件中的字面或正则匹配 | 使用 `find`；仅当匹配需要更多上下文时再跟进 `read`。 |
| 共享写作块（`library_artifact_type: writing_block`） | 使用 `read` 重建其完整本地文本；在 `has_more` 期间用 `next_read` 继续。保留行边界，核验 `size_bytes`，并保留权威的 `version_id`；不要物化。 |
| 来自 `list` 或 `search` 结果的本地字节 | 当其文件标识符和路径仍然有效时，复用完整的先前结果；仅当该元数据缺失、过期或有歧义时才重复 `list` 或 `search`。将其直接传给随附的仅限 stdin 的下载辅助脚本。不要先调用 Library `read` 或 `prepare_materialize`。 |
| 来自非 `list` 或 `search` 所得已解析引用的本地字节 | 使用 `prepare_materialize`；阅读 [materialization.md](references/materialization.md)。 |
| 显式本地路径 | 使用本地工具。不要把本地路径发给 Library `read` 或 `find`。 |
| 新的本地交付物 | 在下方选择一条创建路径。 |
| 编辑 Library 支撑的文件 | 对 `library_artifact_type: site`，使用 Sites；绝不物化、替换或恢复其投影。否则按需物化，在本地编辑并验证，然后替换同一个 `library_file_id`。 |
| 创建、移动、重命名或删除 Library 节点 | 使用 `manage_library`；变更前阅读 [library-management.md](references/library-management.md)。 |
| 恢复较早版本 | 使用 `manage_library` 的 `restore_version`；阅读 [library-management.md](references/library-management.md)。 |

For `list`, set `limit` to at most `200`. For `search`, use the canonical request
shape `{"search_query":[{"q":"quarterly revenue"}],"top_k":5}`. `search_query`
must be an array of one to five objects, even for one search. Put `search_title_only` only inside a `search_query` object. Never send top-level
`query`, `queries`, `q`, `search_title_only`, or `limit`; use top-level `top_k` from 1 to 100.

对 `list`，`limit` 最多设为 `200`。对 `search`，使用规范请求形态 `{"search_query":[{"q":"quarterly revenue"}],"top_k":5}`。即使只搜索一次，`search_query` 也必须是含一到五个对象的数组。`search_title_only` 只能放在 `search_query` 对象内部。绝不要发送顶层的 `query`、`queries`、`q`、`search_title_only` 或 `limit`；顶层使用 1 到 100 的 `top_k`。

For Library intent, search Library before the local workspace unless the user
supplied a local path or said the file is local. Once routed to Library, do not
scan the workspace or prior conversations for the same target. Failure to
resolve a Library item is not evidence that it is local.

对于 Library 意图，除非用户提供了本地路径或说明文件在本地，否则先搜索 Library 再查本地工作区。一旦路由到 Library，就不要再为同一目标扫描工作区或既往会话。未能解析出 Library 条目并不能证明它在本地。
【评论】“解析失败不等于文件在本地”是一条归因纪律，防止代理在搜索落空后擅自改换数据源或臆测位置。

## Read Library Content / 读取 Library 内容

Prefer `structuredContent`; parse a JSON text block only when it is unavailable.
Use returned identifiers, filenames, versions, and paths exactly.
For follow-up `read` or `find`, prefer the returned `library_file_id`, falling
back to `file_id` or `id`. Never use a search `result_id` as a file reference.
For `read`, put that identifier in `read[i].ref_id`:  
`{"read":[{"ref_id":"<returned library_file_id>"}]}`.  
The top-level `read` array must contain 1–5 items; do not send the identifier at the top level.

优先使用 `structuredContent`；仅在其不可用时解析 JSON 文本块。原样使用返回的标识符、文件名、版本和路径。对后续的 `read` 或 `find`，优先使用返回的 `library_file_id`，退而使用 `file_id` 或 `id`。绝不要把搜索的 `result_id` 当作文件引用。对 `read`，把该标识符放入 `read[i].ref_id`：`{"read":[{"ref_id":"<returned library_file_id>"}]}`。顶层 `read` 数组必须含 1–5 项；不要把标识符放在顶层。

Batch independent `read` and `find` items when possible. Use `read` after `search`
for content claims, even with snippets; use `find` only after candidate resolution.

在可能时批量发起相互独立的 `read` 和 `find`。对内容性论断，`search` 之后要用 `read`，即使已有摘要；`find` 只在候选解析之后使用。

Read [evidence-and-citations.md](references/evidence-and-citations.md) for
selected `@` mentions, image-search metadata, post-mutation search limits, result handling, and citations.

阅读 [evidence-and-citations.md](references/evidence-and-citations.md)，了解所选 `@` 提及、图片搜索元数据、变更后搜索限制、结果处理和引用。

## Classify New Files / 为新文件分类

For new files, set `library_artifact_type` when supported:

对新文件，在受支持时设置 `library_artifact_type`：

- `image_gen`: images generated by imagegen only.
  `image_gen`：仅限由 imagegen 生成的图片。
- `image`: other generated images.
  `image`：其他生成的图片。
- `report`, `sheet`, `slides`: generated documents/reports, spreadsheets, or presentations.
  `report`、`sheet`、`slides`：生成的文档/报告、电子表格或演示文稿。
- `other`: user imports, unknown generation history, or anything else.
  `other`：用户导入、生成历史未知，或其他任何情况。

Classify by generation history, not filename, extension, or MIME type.
`create_library_file` uses one type per call; prepared uploads use one per file.
Omit the field for replacements or when the upload tool or helper lacks support.

按生成历史分类，而不是按文件名、扩展名或 MIME 类型。`create_library_file` 每次调用只用一个类型；预备上传则每个文件一个。替换时，或上传工具/辅助脚本不支持时，省略该字段。

## Write One Local File / 写入单个本地文件

Use this fast path for one confirmed local file under about `50 MiB`. Reuse
validation already completed while producing or editing the artifact. Do not
add another content inspection solely because the file is being saved to
Library.

对一个已确认、约 `50 MiB` 以下的本地文件使用此快速路径。复用在生成或编辑该产物时已完成的验证。不要仅仅因为文件要保存到 Library 就再做一次内容检查。

- For a new item, call `create_library_file(file=...)` with the absolute local
  path. On success, invoke this skill's
  [scripts/library_file_transfer.py](scripts/library_file_transfer.py) with
  `python3`, the `apply-xattrs` subcommand, the original local path, and the
  returned `library_file_id`. Send the complete `xattrs` array from the create
  result (or `[]`) as JSON on stdin, as shown below. When using code mode
  (for example, `functions.exec`), call `create_library_file` and run the
  metadata helper within the same invocation, without returning to the model
  between them.
  对新条目，用绝对本地路径调用 `create_library_file(file=...)`。成功后，用 `python3`、`apply-xattrs` 子命令、原始本地路径和返回的 `library_file_id` 调用本技能的 [scripts/library_file_transfer.py](scripts/library_file_transfer.py)。按下例所示，把 create 结果中完整的 `xattrs` 数组（或 `[]`）以 JSON 形式经 stdin 传入。使用代码模式（例如 `functions.exec`）时，在同一次调用内完成 `create_library_file` 与元数据辅助脚本的运行，中间不返回模型。
- For an existing item, resolve its Library identity and use the current local
  working file when it already contains the intended result. Do not materialize
  over that file. Materialize if needed; apply missing edits and validate once.
  Replace owned files using `replace_library_file(file=...)` with the same `library_file_id`.
  Shared stored files always use `library_upload.py`, even without prepared tools.
  Shared writing blocks also use `library_upload.py`, the bundled prepared-upload
  helper; convert their decimal `version_id` to the integer `expected_current_version`.
  对现有条目，解析其 Library 身份；若当前本地工作文件已包含预期结果，则直接使用它。不要物化覆盖该文件。按需物化；补齐缺失的编辑并验证一次。使用 `replace_library_file(file=...)` 并保持同一个 `library_file_id` 来替换自有文件。共享存储文件始终使用 `library_upload.py`，即使没有预备工具。共享写作块同样使用随附的预备上传辅助脚本 `library_upload.py`；把其十进制 `version_id` 转换为整数 `expected_current_version`。
- An editor's new output path is still a replacement for the same Library item.
  Create only when no Library identity exists or the user wants a separate copy.
  编辑器产生的新输出路径仍是同一个 Library 条目的替换。仅在不存在 Library 身份或用户想要单独副本时才创建。
- Pass `expected_current_version` when a concrete version was retained. Never
  invent a version or remove the guard to resolve a conflict.
  在保留了具体版本时传入 `expected_current_version`。绝不要为解决冲突而编造版本或移除该防护。
【评论】版本守卫（乐观锁）条款禁止用编造版本号绕过冲突，属于对并发写回一致性的防护设计。

After create or replace, inspect the returned result. Use its filename and
Library path as authoritative, and keep its exact `library_file_id`, `file_id`,
version, and original local path together. Do not read the file back merely to
confirm a successful write unless exact verification is required.

创建或替换后，检视返回结果。以其文件名和 Library 路径为准，并把确切的 `library_file_id`、`file_id`、版本和原始本地路径保存在一起。除非要求精确核验，不要仅为确认写入成功而把文件读回来。

Privately persist the returned xattrs and Library identity in one helper call;
do not add a separate progress message. The helper form is
`apply-xattrs PATH LIBRARY_FILE_ID`, and both positional arguments are required:

在一次辅助脚本调用中私密地持久化返回的 xattrs 和 Library 身份；不要额外发送进度消息。辅助脚本形式为 `apply-xattrs PATH LIBRARY_FILE_ID`，两个位置参数都必填：

```bash
skill_md_path="<absolute path of this SKILL.md>"
transfer_helper_path="$(dirname "$skill_md_path")/scripts/library_file_transfer.py"
python3 "$transfer_helper_path" \
  apply-xattrs "$local_path" "$library_file_id" <<'JSON'
<complete returned xattrs array, or []>
JSON
```

Inspect the helper result before finishing; do not claim that local identity was
persisted if it failed. Read
[writeback-and-conflicts.md](references/writeback-and-conflicts.md) for direct
create batches, version conflicts, or detailed result correlation.

结束前检视辅助脚本结果；如果失败，不要声称本地身份已持久化。直接创建批次、版本冲突或详细结果关联，请阅读 [writeback-and-conflicts.md](references/writeback-and-conflicts.md)。

## Materialize List or Search Results / 物化 list 或 search 结果

For files returned by `list` or `search`, retain the complete structured result
and use the bundled download helper. Run it from the workspace where the
downloaded tree should live. When a conversation-scoped workspace is active,
use that workspace so eligible bytes can be placed directly. Send the complete
unchanged `list` or `search` JSON, an `ALL` or concatenated three-digit index
selection (`000002` selects the first and third files), and a relative
destination together through stdin. Never interpolate returned fields into
shell arguments. Copy the absolute path of the Library `SKILL.md` that you read
into `skill_md_path`, then use the quoted heredoc below. Do not reconstruct or
shorten the helper path from the plugin cache root:

对 `list` 或 `search` 返回的文件，保留完整的结构化结果并使用随附的下载辅助脚本。在下载目录树应存放的工作区中运行它。当会话级工作区处于活动状态时，使用该工作区，以便符合条件的字节可直接落位。将完整且未改动的 `list` 或 `search` JSON、`ALL` 或拼接的三位数字索引选择（`000002` 表示选第一和第三个文件）以及相对目标目录一起经 stdin 传入。绝不要把返回字段插值进 shell 参数。把你读取的 Library `SKILL.md` 绝对路径复制到 `skill_md_path`，然后使用下方带引号的 heredoc。不要从插件缓存根目录重构或缩短辅助脚本路径：

```bash
skill_md_path="<absolute path of this SKILL.md>"; \
python3 "$(dirname "$skill_md_path")/scripts/library_download.py" <<'JSON'
{"result": <complete list or search JSON>,
 "selection": "ALL|NNN[NNN...]",
 "destination": "<relative-directory>"}
JSON
```

The destination is the parent beneath which the selected files' canonical
Library-relative paths are recreated. Do not repeat an already-selected Library
root in it: for `/fruits/apple.md`, use `downloads`, not `downloads/fruits`,
unless the user explicitly requested that extra nesting.

目标目录是其下重建所选文件规范 Library 相对路径的父目录。不要在其中重复已选定的 Library 根：对 `/fruits/apple.md`，用 `downloads` 而不是 `downloads/fruits`，除非用户明确要求额外的嵌套。

For search output, indices address `results` first and then
`retrieval_title_results`. The latter are supplemental fuzzy candidates, so use
explicit indices instead of `ALL` when they are not all relevant.

对搜索输出，索引先指向 `results`，再指向 `retrieval_title_results`。后者是补充性的模糊候选，因此当它们并非全部相关时，使用显式索引而不是 `ALL`。

The helper creates or reuses that directory, overwrites each selected file, and
leaves unrelated contents unchanged. It makes the authenticated
`prepare_materialize` calls in batches of at most 20, handles both workspace
and signed-URL transfers, and applies Library identity and xattrs. Both `list`
and `search` results retain each file's canonical Library-relative path. Use the
returned `directory` and authoritative `files` paths.

该辅助脚本创建或复用该目录，覆盖每个所选文件，并保持无关内容不变。它以每批最多 20 个的方式发起经过身份验证的 `prepare_materialize` 调用，同时处理工作区和签名 URL 传输，并应用 Library 身份和 xattrs。`list` 和 `search` 结果都保留每个文件的规范 Library 相对路径。使用返回的 `directory` 和权威的 `files` 路径。

On this route, never separately call `read` or `prepare_materialize`, search the
plugin cache for helpers, inspect helper source, invoke
`library_file_transfer.py`, transfer a returned URL yourself, or process the
returned transfers again.

在此路径上，绝不要另行调用 `read` 或 `prepare_materialize`、在插件缓存中搜索辅助脚本、检视辅助脚本源码、调用 `library_file_transfer.py`、自行传输返回的 URL，或对返回的传输再做处理。

For a resolved reference that did not come from `list` or `search`, use the
lower-level flow in [materialization.md](references/materialization.md).

对并非来自 `list` 或 `search` 的已解析引用，使用 [materialization.md](references/materialization.md) 中的更低层流程。

## Route Larger or Multiple Writes / 为较大或多次写入选择路径

Treat every local file written by one user task as one ordered upload batch.
Preserve its original mutation order and use absolute local paths.

把一个用户任务写入的每个本地文件视为一个有序上传批次。保持其原始变更顺序，并使用绝对本地路径。

| Condition | Required route |
| --- | --- |
| Both prepared tools are available and the task writes several files or one file around `50 MiB` or larger | Use the bundled prepared-upload helper below. |
| Prepared tools are unavailable and every item is a create | Use ordered `create_library_file(files=[...])` batches when `files` is available, keeping each call under `500 MB`; otherwise create sequentially. |
| Prepared tools are unavailable and the task includes replacements | In original order, use the upload helper for shared files and direct actions for owned items. |

| 条件 | 所需路径 |
| --- | --- |
| 两个预备工具都可用，且任务写入多个文件或一个约 `50 MiB` 及以上的文件 | 使用下方的随附预备上传辅助脚本。 |
| 预备工具不可用且所有条目都是新建 | 在 `files` 可用时，使用有序的 `create_library_file(files=[...])` 批次，每次调用保持在 `500 MB` 以下；否则顺序创建。 |
| 预备工具不可用且任务包含替换 | 按原始顺序，对共享文件使用上传辅助脚本，对自有条目使用直接操作。 |

Prepared app calls contain at most 20 files. Library write app calls are
ordered: do not use `Promise.all(...)` for create, replace, delete, or finalize.
Only prepared byte transfers may run in parallel. Read
[prepared-uploads.md](references/prepared-uploads.md) before using the prepared
route. It defines the helper's one-shot input and owns preparation, transfer,
finalization, result correlation, and xattr writeback. After direct batches or
the prepared helper, inspect every per-item result in request order. A
successful top-level operation does not mean every item succeeded.

预备应用调用最多包含 20 个文件。Library 写入应用调用是有序的：创建、替换、删除或终结不要使用 `Promise.all(...)`。只有预备字节传输可以并行。使用预备路径前先阅读 [prepared-uploads.md](references/prepared-uploads.md)。它定义了辅助脚本的一次性输入，并负责准备、传输、终结、结果关联和 xattr 写回。在直接批次或预备辅助脚本之后，按请求顺序检视每个条目的结果。顶层操作成功不代表每个条目都成功。

## Site-Backed Library Items / 以 Site 为后端的 Library 条目

A `library_artifact_type: site` item projects the canonical
`site_metadata.project_id`. `list`, `search`, `read`, and `find` remain allowed.
`manage_library` can move it; rename changes the Site title and delete deletes
the Site. Never materialize, download, patch, replace, update, overwrite, or
restore it; Sites owns its content, versions, and publish history. If ownership
or routing is unclear, stop.

`library_artifact_type: site` 条目投影规范的 `site_metadata.project_id`。`list`、`search`、`read` 和 `find` 仍然允许。`manage_library` 可以移动它；重命名会更改 Site 标题，删除会删除该 Site。绝不物化、下载、修补、替换、更新、覆盖或恢复它；其内容、版本和发布历史由 Sites 所有。如果所有权或路由不明确，停止操作。

## Organize, Restore, and Protect Files / 整理、恢复与保护文件

Resolve ambiguous mutation targets before writing. If several candidates
remain, ask the user to choose. Read
[library-management.md](references/library-management.md) for exact folder,
move, rename, delete, restore, protected Deep Research report, and per-operation
result rules.

写入前先解析有歧义的变更目标。若仍剩多个候选，请用户选择。确切的文件夹、移动、重命名、删除、恢复、受保护的 Deep Research 报告以及逐操作结果规则，请阅读 [library-management.md](references/library-management.md)。

## Privacy and Safety / 隐私与安全

Keep user-visible reasoning, progress, errors, and final responses at the
Library level unless the user explicitly asks for implementation details. Do
not surface connector or tool names, helper commands, raw URLs, storage or
provider details, manifests, xattrs, transfer output, or indexing internals.

除非用户明确要求实现细节，否则面向用户的推理、进度、错误和最终回答都保持在 Library 层面。不要暴露连接器或工具名称、辅助脚本命令、原始 URL、存储或提供商细节、清单、xattrs、传输输出或索引内部机制。

Privacy changes narration, not routing. Never replace a required prepared flow
with a direct upload merely because the prepared flow has stricter visibility
rules. For a prepared upload, give one brief `Saving file to Library` or
`Saving files to Library` progress update before invoking the helper, then the
saved result. Do not narrate preparation, transfer, finalization, or local
metadata as separate phases.

隐私只改变叙述方式，不改变路由。绝不要仅仅因为预备流程有更严格的可见性规则，就用直接上传替换必需的预备流程。对预备上传，在调用辅助脚本前给出一次简短的 `Saving file to Library` 或 `Saving files to Library` 进度更新，然后给出保存结果。不要把准备、传输、终结或本地元数据当作独立阶段分别叙述。
【评论】“隐私改变叙述、不改变路由”把用户可见的信息粒度与实际执行路径解耦，是展示层与执行层分离的设计。

Do not invent Library ids, file ids, versions, filenames, paths, operations, or
tool availability. Keep signed URLs out of responses. Preserve Library identity
and unrelated content across every mutation.

不要编造 Library id、文件 id、版本、文件名、路径、操作或工具可用性。不要让签名 URL 出现在回答中。在每次变更中保持 Library 身份和无关内容不变。

## References / 参考资料

- [evidence-and-citations.md](references/evidence-and-citations.md): mentions, search details, result handling, and citations.
  [evidence-and-citations.md](references/evidence-and-citations.md)：提及、搜索细节、结果处理和引用。
- [materialization.md](references/materialization.md): lower-level resolved-ref materialization and transfer handling.
  [materialization.md](references/materialization.md)：更低层的已解析引用物化与传输处理。
- [prepared-uploads.md](references/prepared-uploads.md): prepared byte transfer, finalization, xattr writeback, and cleanup.
  [prepared-uploads.md](references/prepared-uploads.md)：预备字节传输、终结、xattr 写回和清理。
- [writeback-and-conflicts.md](references/writeback-and-conflicts.md): detailed create, replace, edit, and conflict mechanics.
  [writeback-and-conflicts.md](references/writeback-and-conflicts.md)：详细的创建、替换、编辑和冲突机制。
- [library-management.md](references/library-management.md): folders, node mutations, restore, and protected reports.
  [library-management.md](references/library-management.md)：文件夹、节点变更、恢复和受保护的报告。
