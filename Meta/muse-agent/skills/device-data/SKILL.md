---
name: "device_data"
title: "Device Data"
description: "Read cached contacts and calendar events from Muse storage. Delete Muse's local copy of either source without modifying paired devices."
metadata: { "includeInPrompt": true }
---

<!-- BILINGUAL-EN-ZH -->
# Device Data / 设备数据

## Purpose / 用途

Read cached contacts and calendar events directly from Muse storage, without
contacting the device. Use it when a live read is not possible: the device is
offline, or it does not offer `contacts.search`/`calendar.search`. Delete all
locally stored contacts or calendar data when the user explicitly requests it.
The tool does not contact or mutate a device.

直接从 Muse 存储读取缓存的联系人与日历事件，不联系设备。在无法实时读取时使用：设备离线，或不提供 `contacts.search`/`calendar.search`。当用户明确请求时，删除所有本地存储的联系人或日历数据。该工具不联系也不改动设备。

## Commands / 命令

```bash
device-data --help
device-data contacts status
device-data contacts --delete-all
device-data contacts search --match-mode ranked \
                            [--query <text> [--phone-label <text>] \
                             [--locale <bcp-47>]] \
                            [--device <node_id>] [--limit <n>] [--offset <n>]
device-data calendar search [--query <text>] [--device <node_id>] \
                            [--start-date <YYYY-MM-DD>] \
                            [--end-date <YYYY-MM-DD>]
device-data calendar --delete-all
```

Both searches default to every eligible paired device. Use `--device` to scope
to one and omit `--query` to list.

两种搜索默认覆盖所有符合条件的已配对设备。用 `--device` 限定到一台，省略 `--query` 则为列举。

Contact search combines literal display-name, phone, and email substring
matches with token-level exact, locale-aware personal-name nickname, and
phonetic name matching. With no locale or an English locale, ranked search
applies OACR-style fuzzy matching only after ordinary name matching returns
nothing for that source device. It compares cleaned query tokens of at least
four characters against the cleaned full name and canonical indexed name
tokens, including simplified name variants, allows one edit, and returns at
most three fuzzy candidates per source device. Explicit non-English locales
retain the existing bounded Unicode full-name edit-distance fallback.
Ranked name evidence feeds the downstream call-authorization policy. Fuzzy,
partial, and phonetic evidence must pass bounded token revalidation before a
complete singleton autodials; literal matches never autodial. Nickname
singletons rely on the explicit locale-selected nickname map. Weak, ambiguous,
incomplete, or unanchored matches still require confirmation.

联系人搜索把显示名、电话与电子邮件的字面子串匹配，与词元级精确匹配、感知区域设置的个人姓名昵称匹配以及音韵姓名匹配相结合。在无区域设置或英语区域设置下，排序搜索只有在该源设备的普通姓名匹配无结果时才应用 OACR 风格的模糊匹配。它把至少四个字符的清洗后查询词元与清洗后的全名及规范索引姓名词元（包括简化姓名变体）比较，允许一次编辑，每个源设备最多返回三个模糊候选。明确的非英语区域设置保留既有的有界 Unicode 全名编辑距离回退。排序姓名证据会输入下游的通话授权策略。模糊、部分与音韵证据必须先通过有界词元复验，完整的唯一匹配才能自动拨号；字面匹配从不自动拨号。昵称唯一匹配依赖显式按区域设置选择的昵称映射。微弱、含糊、不完整或未锚定的匹配仍需确认。

Ranked results also include `selection_evidence`. Its
`spoken_name_collision_pairs` identifies differently spelled plausible matches
whose full names have strong phonetic coverage in both directions; a nonempty
list sets `requires_spoken_name_clarification`, because spoken input has not
selected between them. Collision entries carry a comma-separated
`spoken_spelling` suitable for distinct TTS pronunciation in either a call or
message clarification.
Each contact includes `phone_selection`, which groups safely equivalent
formatting variants into one destination and reports
`distinct_usable_phone_count` and
`requires_phone_clarification`. When `--phone-label` is present,
`requested_label_match_count` reports how many distinct destinations carry
that label or a recognized equivalent such as `cell` and `mobile`. Each option
includes a TTS-safe `spoken_suffix` for the rare case where labels do not
distinguish the saved lines.

排序结果还包含 `selection_evidence`。其 `spoken_name_collision_pairs` 识别拼写不同、但全名在双向都具有强音韵覆盖的合理匹配；该列表非空时设置 `requires_spoken_name_clarification`，因为语音输入尚未在它们之间做出选择。碰撞条目携带逗号分隔的 `spoken_spelling`，适用于通话或消息澄清中可区分的 TTS 发音。每个联系人包含 `phone_selection`，它把安全等价的格式变体归为一个目的地，并报告 `distinct_usable_phone_count` 与 `requires_phone_clarification`。当提供 `--phone-label` 时，`requested_label_match_count` 报告有多少个不同目的地带有该标签或被认可的等价标签，如 `cell` 与 `mobile`。每个选项包含 TTS 安全的 `spoken_suffix`，用于标签无法区分所存号码的罕见情形。

Compound given names match as a unit: under the English default, `Mary Ann`
can find a contact named `Mary`. Pass `--locale` to select the
French, Italian, or Spanish nickname map when the user's locale is known; for
example, `Mamen` finds `María del Carmen` with `--locale es`, and `Coco` finds
`Jean Claude` with `--locale fr`. Language subtags `ar`, `ja`, `zh`, and `ko`
disable nickname matching; every other locale without a dedicated map uses the
English fallback. If the locale is unavailable, omit the flag to use that
English nickname, fuzzy, and spoken-emoji default. Queries containing Chinese,
Japanese, or Korean characters retain the bounded whole-name edit-distance
fallback without requiring locale plumbing. Explicit non-English locales do not
inherit English spoken-emoji labels. Phone matching ignores formatting
and country-code differences when the final seven digits agree, but only for a  
phone-shaped  
query; digits embedded in names or emails do not activate phone matching.
Ranked name search is order-independent, treats apostrophes and
hyphens symmetrically, tolerates repeated or unmatched query words, and reports
coarse `match` evidence (`class`, matched/total tokens, and unmatched tokens).
Results are a single ranked, paginated list; they do not assert that one contact
has been selected. Pass an explicitly requested phone type through
`--phone-label` rather than including it in the name query. Phone numbers are
ordered by the requested label first, then mobile, home, work, other, and custom
labels. Results report `phone_label_matched`; a label miss does not exclude the
contact.
Organization and job title are included when available. Ranked results can be
paged by advancing `offset` by the returned `count` while `has_more` is true.
`search_complete` is false when the scoped contact corpus, bounded
preprocessing, or a ranked result bound overflows, independently from
pagination; inspect
`corpus_overflow`, `preprocessing_overflow`, and `result_overflow` for the
cause. `fuzzy_overflow` separately reports that English fuzzy recovery hit its
non-pageable, per-source confirmation cap; it does not make an otherwise exact
result incomplete.

复合名字作为整体匹配：在英语默认下，`Mary Ann` 可以找到名为 `Mary` 的联系人。当用户的区域设置已知时，传 `--locale` 选择法语、意大利语或西班牙语昵称映射；例如 `Mamen` 用 `--locale es` 找到 `María del Carmen`，`Coco` 用 `--locale fr` 找到 `Jean Claude`。语言子标签 `ar`、`ja`、`zh` 与 `ko` 禁用昵称匹配；其他没有专用映射的区域设置使用英语回退。若区域设置不可用，省略该标志以使用英语昵称、模糊与语音表情默认。包含中文、日文或韩文字符的查询保留有界的整名编辑距离回退，无需区域设置管道。明确的非英语区域设置不继承英语语音表情标签。当末七位数字一致时，电话匹配忽略格式与国家码差异，但仅对电话形状的查询生效；嵌入姓名或电子邮件中的数字不会激活电话匹配。排序姓名搜索与词序无关，对撇号与连字符对称处理，容忍重复或未匹配的查询词，并报告粗粒度的 `match` 证据（`class`、匹配/总词元数与未匹配词元）。结果是单一的排序分页列表；它们不断言已选定某个联系人。明确请求的电话类型通过 `--phone-label` 传入，而不是包含在姓名查询中。电话号码先按请求的标签排序，再按 mobile、home、work、other 与自定义标签排序。结果报告 `phone_label_matched`；标签未命中不排除该联系人。组织与职位在可得时包含。排序结果可通过在 `has_more` 为真时按返回的 `count` 推进 `offset` 来分页。当限定范围的联系人语料、有界预处理或排序结果上限溢出时，`search_complete` 为假，与分页无关；检查 `corpus_overflow`、`preprocessing_overflow` 与 `result_overflow` 寻找原因。`fuzzy_overflow` 单独报告英语模糊恢复达到其不可分页的每源确认上限；它不会使本来精确的结果变为不完整。

## Operating Rules / 操作规则

1. Read fresh this turn; never answer a contacts or calendar question from memory.
   本回合读取最新数据；绝不凭记忆回答联系人或日历问题。
2. Treat device data as private user data: show only the fields the request needs, and do not infer sensitive attributes from contacts, locations, device names, or communication metadata.
   把设备数据当作私人用户数据：只展示请求所需字段，不要从联系人、位置、设备名称或通信元数据推断敏感属性。
   【评论】这是一条数据最小化与隐私条款：限制展示字段，并明确禁止从通信元数据推断敏感属性。
3. Use inclusive `YYYY-MM-DD` bounds for calendar dates, with the same date for
   a single day; all-day result `end_date` values are also inclusive. Read event
   times from the tagged `time` object: `timed` keeps the existing localized
   `start_at` / `end_at` fields and adds their UTC forms plus `user_timezone`,
   the IANA display zone used to render those local values; `all_day` has
   date-only values that must not be timezone-shifted. Both carry
   `start_weekday` / `end_weekday`, already resolved against the value beside
   them; use those and never work a weekday out from the date yourself.
   Contact reads likewise preserve `synced_at` and add semantic
   `contact_synced_at` or `contact_sync_completed_at` UTC/user-local forms.
   日历日期使用含端点的 `YYYY-MM-DD` 边界，单日用同一日期；全天结果的 `end_date` 也是含端点的。从带标签的 `time` 对象读取事件时间：`timed` 保留既有的本地化 `start_at` / `end_at` 字段，并添加其 UTC 形式以及 `user_timezone`（用于渲染这些本地值的 IANA 显示时区）；`all_day` 只有日期值，绝不能做时区平移。两者都携带 `start_weekday` / `end_weekday`，已相对旁边的值解析好；使用它们，绝不自己从日期推算星期几。联系人读取同样保留 `synced_at`，并添加语义化的 `contact_synced_at` 或 `contact_sync_completed_at` UTC/用户本地形式。
4. Treat an empty calendar result as authoritative only when `cache.coverage`
   is `complete`. For contacts, `search_complete` covers the bounded corpus and
   ranked result sets, not source freshness; consult contacts status when
   freshness matters and report partial or truncated results.
   仅当 `cache.coverage` 为 `complete` 时，才把空的日历结果当作权威。对联系人，`search_complete` 覆盖有界语料与排序结果集，而非来源新鲜度；当新鲜度重要时查询 contacts status，并报告部分或截断的结果。
5. If no record exists, say so. A device calendar is only that phone's view;
   check other connected calendars when relevant.
   如果没有记录，就直说。设备日历只是那部手机的视图；相关时检查其他已连接的日历。
6. When the user asks to delete contacts or calendar events without naming the
   target, do not infer the target. Confirm that the user means Muse's locally
   stored copy before deleting anything.
   当用户要求删除联系人或日历事件但未指明目标时，不要推断目标。删除任何东西之前，确认用户指的是 Muse 本地存储的副本。
   【评论】删除类请求被要求显式确认目标且禁止推断，属于破坏性操作的确认设计；配合规则 7 的"只跑对应命令、不枚举记录 ID"进一步收窄操作面。
7. After the user explicitly requests deletion of Muse's locally stored copy,
   run the command that matches the source:  
   `device-data contacts --delete-all` or  
   `device-data calendar --delete-all`.  
   Do not enumerate record IDs.
   在用户明确请求删除 Muse 本地存储的副本之后，运行与来源匹配的命令：  
   `device-data contacts --delete-all` 或  
   `device-data calendar --delete-all`。  
   不要逐一列举记录 ID。
8. Both `--delete-all` commands delete the selected source from Muse storage
   across all devices. They delete cached records, source-ledger payloads,
   upload staging, and sync state. They preserve device pairing and every other
   data source. A later device sync can store new records for the deleted
   source.
   两条 `--delete-all` 命令都会跨所有设备从 Muse 存储中删除所选来源。它们删除缓存记录、来源账本负载、上传暂存与同步状态。它们保留设备配对与所有其他数据来源。之后的设备同步可以为被删来源存入新记录。
