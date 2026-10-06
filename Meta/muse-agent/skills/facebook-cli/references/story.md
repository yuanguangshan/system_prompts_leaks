<!-- BILINGUAL-EN-ZH -->
# Facebook Stories / Facebook 快拍

Fetch your story feed (story tray) to see recent stories from friends and pages you follow.

拉取你的快拍信息流（story tray），查看好友与你所关注页面的近期快拍。

## Commands / 命令

### Fetch Story Feed / 拉取快拍信息流

```bash
facebook-cli story feed [--limit <n>]
```

**Options:** / **选项：**
- `--limit` (optional): Maximum number of story buckets to return (default 20, max 50)
  `--limit`（可选）：返回的快拍分组的最大数量（默认 20，最大 50）

**Examples:** / **示例：**
```bash
# Fetch your story feed
facebook-cli story feed

# Fetch a smaller set
facebook-cli story feed --limit 5
```

**Response fields:** / **响应字段：**

Each story bucket represents stories from one author:

每个快拍分组代表同一位作者的若干快拍：

- `owner_name`: Name of the story author
  `owner_name`：快拍作者的名称
- `owner_id`: Profile ID of the story author
  `owner_id`：快拍作者的个人主页 ID
- `seen`: Whether the bucket has been viewed
  `seen`：该分组是否已被查看
- `cards`: Array of individual story cards, each containing:
  `cards`：由单条快拍卡片组成的数组，每张卡片包含：
  - `story_id`: Unique story ID
    `story_id`：快拍的唯一 ID
  - `creation_time`: When the story was posted (Unix timestamp)
    `creation_time`：快拍发布时间（Unix 时间戳）
  - `expiry_time`: When the story expires (Unix timestamp)
    `expiry_time`：快拍过期时间（Unix 时间戳）
  - `message`: Story text content (if any)
    `message`：快拍文字内容（如有）
  - `story_url`: Direct link to the story
    `story_url`：快拍的直接链接
