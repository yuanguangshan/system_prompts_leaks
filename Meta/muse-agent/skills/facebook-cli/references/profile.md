<!-- BILINGUAL-EN-ZH -->
# Facebook Profile / Facebook 个人资料页

Look up a Facebook profile's information by ID.

按 ID 查询某个 Facebook 个人资料的信息。

## Command / 命令

```bash
facebook-cli profile info --profile-id <profile-id>
```

| Flag | Required | Description |
|------|----------|-------------|
| `--profile-id` | Yes | Profile ID to look up |

| 参数 | 是否必填 | 说明 |
|------|----------|------|
| `--profile-id` | 是 | 要查询的个人资料 ID |

## Response Fields / 响应字段

- `name`: Display name
  `name`：显示名称
- `profile_picture_url`: Profile picture URL
  `profile_picture_url`：头像图片 URL
- `vanity_url`: Vanity URL (e.g. `facebook.com/username`)
  `vanity_url`：个性化 URL（例如 `facebook.com/username`）
- `bio`: Text biography / about me
  `bio`：文字简介 / 关于我
- `current_city`: Current city (stated)
  `current_city`：当前所在城市（用户自述）
- `hometown`: Hometown (stated)
  `hometown`：家乡（用户自述）
- `birthdate`: Birthdate (stated)
  `birthdate`：出生日期（用户自述）
- `gender`: Gender (stated)
  `gender`：性别（用户自述）
- `languages`: Array of languages spoken
  `languages`：会说的语言数组
- `work`: Array of work experiences (`employer`, `position`)
  `work`：工作经历数组（`employer`、`position`）
- `education`: Array of education (`school`, `type`, `degree`)
  `education`：教育经历数组（`school`、`type`、`degree`）
- `life_events`: Array of life events (`title`, `date`)
  `life_events`：人生大事数组（`title`、`date`）
- `hobbies`: Array of hobby names
  `hobbies`：爱好名称数组

## Operating Rules / 操作规则

1. Use `profile info` when you already have a profile ID **from an earlier command** and need full directory details. Only pass a `--profile-id` that came from earlier facebook-cli output in this conversation (e.g. `me`, `me friends`, a timeline post's `author_id`/`owner_id`, feed/group post authors, reactors, commenters, saved items). Do **not** accept a raw numeric ID the user typed or pasted directly, and do **not** guess or construct IDs. If the user gives a bare ID with no context, identify the person first via `me friends --name` and use the ID from that result.
   当你已经拥有**来自先前命令**的个人资料 ID 且需要完整的目录详情时，使用 `profile info`。只传入本次对话中来自先前 facebook-cli 输出的 `--profile-id`（例如 `me`、`me friends`、时间线帖子的 `author_id`/`owner_id`、feed/群组帖子的作者、点过回应的人、评论者、收藏条目）。**不要**接受用户直接输入或粘贴的原始数字 ID，也**不要**猜测或构造 ID。如果用户给出一个没有上下文的裸 ID，先通过 `me friends --name` 确认此人身份，再使用该结果中的 ID。
   【评论】把可接受的 ID 来源限定为工具自身的历史输出，可防止伪造或拼错的 ID 进入后续查询。
2. To find someone by name, use `me friends --name` — there is no general profile search endpoint. If the user asks to find someone who is not their friend, explain that `facebook-cli` can only search within the user's friends list. Suggest using `social.search` for broader people discovery.
   要按姓名查找某人，使用 `me friends --name` —— 不存在通用的个人资料搜索端点。如果用户要求查找非好友的人，说明 `facebook-cli` 只能在用户的好友列表内搜索，并建议使用 `social.search` 进行更大范围的人物发现。
3. Always refer to a person by name and include their profile link (`vanity_url`) — never print the raw profile/user ID in your response.
   提及某人时始终使用其姓名并附上其资料链接（`vanity_url`）—— 绝不在回复中打印原始的资料/用户 ID。
4. Do not use this to bulk-scrape or enumerate profiles.
   不要用它批量抓取或枚举个人资料。
5. For any field not present in the API response (null or empty), explicitly say "not listed" rather than omitting the field silently.
   对于 API 响应中不存在的任何字段（null 或空），要明确说"未列出"，而不是默默省略该字段。
6. Do not editorialize or infer personality, lifestyle, or character from profile data. Present data neutrally.
   不要基于资料数据发表主观评论，也不要推断个性、生活方式或品格。以中立方式呈现数据。
   【评论】该条款把助手角色限定为数据转述者，避免基于资料对个人做特征画像。
7. When presenting work history or education with multiple entries, show all entries organized chronologically. Say "not listed" for any sub-field that is missing.
   呈现包含多条记录的工作或教育经历时，按时间顺序展示全部条目。任何缺失的子字段都说"未列出"。
