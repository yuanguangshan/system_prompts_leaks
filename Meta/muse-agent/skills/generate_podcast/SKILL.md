---
name: "generate_podcast"
description: "Compose and deliver audio content: a podcast episode, briefing, or narrated summary, with one or more voices, as an MP3. For reading supplied text aloud verbatim, use tts."
metadata: { "includeInPrompt": true }
---
<!-- BILINGUAL-EN-ZH -->
# Generate Podcast / 生成播客

## Purpose / 目的
Generate audio content — podcasts, audio briefings, narrated summaries, or any spoken audio. Use this skill for all generated audio, not just podcasts. The `podcast-helper` script handles synthesis, catalog management, cover art generation, and publishing.

生成音频内容——播客、音频简报、旁白摘要或任何口播音频。所有生成的音频都使用本技能，而不仅仅是播客。`podcast-helper` 脚本负责合成、目录管理、封面生成与发布。

## Workflow / 工作流

### 1. Plan / 规划
- **Topic and angle**
  **主题与角度**
- **Speaker names and voices** — **prefer the Meta AI voices by default** (the `MAI_01` and `MAI_03` catalog ids: Warm and Smooth); they are the recommended, production-quality set and the right pick most of the time. Default to Warm (`avocado_v2:MAI_01`) and Smooth (`avocado_v2:MAI_03`). When the topic, tone, or user request steers toward a different catalog voice — an accent, a character fit, or more distinct speakers — it's perfectly fine to pick it from the catalog. The full voice catalog is at `/opt/hatch/skills/voice-selector/voice_source.json` — pick by the entry's display `name` and pass its `id` (colon form like `avocado_v2:MAI_01` is fine). Any number of speakers is supported. ("MAI" is the internal id prefix only — say "Meta AI voices" or display names to the user, never "MAI".)
  **说话人姓名与嗓音**——**默认优先使用 Meta AI 嗓音**（目录 id `MAI_01` 与 `MAI_03`：Warm 和 Smooth）；它们是推荐的、达到生产质量的组合，多数情况下是正确选择。默认用 Warm（`avocado_v2:MAI_01`）和 Smooth（`avocado_v2:MAI_03`）。当主题、语气或用户请求指向目录中别的嗓音——某种口音、更贴合的角色、或区分度更强的说话人——从目录中挑选完全没问题。完整的嗓音目录在 `/opt/hatch/skills/voice-selector/voice_source.json`——按条目的展示名 `name` 挑选并传入其 `id`（冒号形式如 `avocado_v2:MAI_01` 即可）。支持任意数量的说话人。（"MAI" 只是内部 id 前缀——对用户要说 "Meta AI voices" 或展示名，绝不说 "MAI"。）
- **Host names** — give each speaker a real first name (e.g. Alex, Jordan). These are the show's own character names, chosen by you and separate from the voice display names: never label a speaker with a voice name like "Warm" or "Smooth". **Remember the name you give each voice** — record the pairing in `~/MEMORY.md` (e.g. "podcast host Alex = avocado_v2:MAI_01") and, before naming a new host, check there for a name you've already used for that voice. Reuse the same name whenever you use that voice again, so a given voice is always the same character to the user. For a **recurring series**, reuse the same host names and the same voice assignments in every episode so it sounds like the same show — pin them in the cron task body (see Scheduling).
  **主持人姓名**——给每位说话人一个真实的名（如 Alex、Jordan）。这些是节目自己的角色名，由你选定，并与嗓音展示名相互独立：绝不用嗓音名（如 "Warm" 或 "Smooth"）标注说话人。**记住你给每个嗓音起的名字**——把配对记录在 `~/MEMORY.md`（如 "podcast host Alex = avocado_v2:MAI_01"），并且在给新主持人起名前，先到那里查一下该嗓音是否已有用过的名字。再次使用同一嗓音时复用同一名字，让同一嗓音对用户而言始终是同一角色。对**常设系列**，每期都复用相同的主持人姓名与相同的嗓音分配，让它听起来是同一档节目——把它们固定写在 cron 任务体里（见 Scheduling 一节）。
- **Target length** — 5–15 minutes; ~130 words/minute
  **目标时长**——5–15 分钟；约每分钟 130 词
- **Avoid repeating past episodes** — before settling the topic and angle, run `podcast-helper manifest read` and look at each recent episode's `topics` field (a short summary of what that episode already covered). Steer the new episode toward fresh material or a genuinely new angle rather than re-covering ground. This matters most for a recurring series.
  **避免重复过去的节目**——在敲定主题与角度之前，先运行 `podcast-helper manifest read`，查看最近每期节目的 `topics` 字段（对该期已覆盖内容的简短摘要）。把新一期引向新材料或真正的新角度，而不是重复已讲过的内容。这一点对常设系列最为重要。

### 2. Write the script / 撰写脚本
Write the full dialogue. Label each turn with the speaker name followed by a colon:  
写出完整对话。每句话以说话人名加冒号标注：  
```
Alex: Welcome to the podcast. Today we're diving into async Rust.
Jordan: Great topic. Let's start with why async matters.
Alex: The big advantage is zero-cost abstractions...
```

Write the script to a file (e.g. `/tmp/script.txt`).

把脚本写入一个文件（如 `/tmp/script.txt`）。

**The script goes directly to a TTS model — write it in spoken form.** The text will be read aloud exactly as written. Follow these guidelines:

**脚本会直接送入 TTS 模型——请以口语形式书写。**文本将按原样逐字朗读。请遵循以下准则：

- **Numbers**: write out in full — "one hundred thousand", "forty-two dollars and fifty cents", "three point five percent"
  **数字**：完整写出——"one hundred thousand"、"forty-two dollars and fifty cents"、"three point five percent"
- **Times**: "ten o'clock A M", "three thirty P M", "noon", "midnight"
  **时间**："ten o'clock A M"、"three thirty P M"、"noon"、"midnight"
- **Years**: "twenty twenty-five", "nineteen ninety-nine"
  **年份**："twenty twenty-five"、"nineteen ninety-nine"
- **Abbreviations**: spell out — "U S A", "A P I" (pronounceable acronyms like "NASA" can stay)
  **缩写**：逐字母拼出——"U S A"、"A P I"（可自然发音的首字母缩略词如 "NASA" 可保留）
- **URLs**: simplify — "example dot com link" instead of raw URLs
  **网址**：简化——用 "example dot com link" 代替原始 URL
- **Email**: "john at company dot com"
  **邮箱**："john at company dot com"
- **Lists/data**: convert to natural sentences, limit to top 3–5 items
  **列表/数据**：转换为自然的句子，最多取前 3–5 项
- **Punctuation**: use commas for natural pauses. If a sentence runs long, split it into two *complete* sentences — do not clip it into fragments.
  **标点**：用逗号表示自然停顿。如果句子过长，拆成两个*完整*的句子——不要截成残缺片段。
- **Never include**: stage directions like `(pause)`, sound effects, markup, emoji, citation numbers like `[1]`, math symbols, or visual separators like `---`
  **绝不包含**：`(pause)` 之类的舞台指示、音效、标记语言、emoji、`[1]` 之类的引文编号、数学符号，或 `---` 之类的视觉分隔符

**Write complete, conversational sentences — not headlines.** Every line a host speaks should be a full grammatical sentence with a subject and a verb, the way people actually talk out loud. Avoid telegraphic, verbless fragments and stat-ticker read-outs; this is a conversation, not a wire report or a scoreboard. An occasional short line for emphasis ("Unbelievable.") is fine, but never string clipped fragments together.

**写出完整的、对话式的句子——而不是标题式短语。**主持人说的每一句都应当是有主语有谓语的完整语法句，就像人们真正开口说话那样。避免电报式的无谓语片段和数据播报式的罗列；这是对话，不是通讯社快讯，也不是记分牌。偶尔来一句强调用的短句（"Unbelievable."）没问题，但绝不要把残缺片段串在一起。

- Avoid: `Second assist of the night for Messi. Two one Argentina. Full time. Heartbreak for England.`
  避免：`Second assist of the night for Messi. Two one Argentina. Full time. Heartbreak for England.`
- Prefer: `That's Messi's second assist of the night, and it puts Argentina up two to one. When the whistle went, it was heartbreak for England.`
  更好：`That's Messi's second assist of the night, and it puts Argentina up two to one. When the whistle went, it was heartbreak for England.`

When you present a stat or a scoreline, fold it into a spoken sentence ("England had just forty-four percent of the ball") rather than dropping it in as a bare fragment ("England, forty-four percent possession").

呈现数据或比分时，把它融进口语句子（"England had just forty-four percent of the ball"），而不是作为干巴巴的片段丢出来（"England, forty-four percent possession"）。

### 3. Generate / 生成

**Write a topics summary for dedup.** Before generating, write a short bulleted summary of the major topics and key points this episode covered to a file (e.g. `/tmp/topics.md`) and pass it with `--topics-file`. Keep it concise — a handful of bullets naming the subjects and any specific stories, guests, or angles, not a full transcript. It is persisted alongside the episode and read by future generations (via the `topics` field in `manifest read`) to avoid repeating material. Example:  
**写一份用于去重的主题摘要。**生成之前，把本期覆盖的主要话题与要点写成简短的要点列表，存入一个文件（如 `/tmp/topics.md`），并用 `--topics-file` 传入。保持简洁——几条要点，点出主题及具体的故事、嘉宾或角度即可，不要写成完整逐字稿。它会随节目一起持久化，供未来的生成读取（通过 `manifest read` 的 `topics` 字段）以避免重复素材。示例：  
```
- Async Rust fundamentals: futures, executors, zero-cost abstractions
- Tokio vs async-std tradeoffs
- Common pitfalls: blocking in async contexts, .await forgetting
```

**Default: generate only, do not publish.** Only add `--publish` when the user explicitly asked to publish or when the podcast catalog already has an RSS feed — a `"feed"` object with a non-empty `feed_url` (meaning they've published to a feed before and want new episodes added). A `"feed"` object that has only a `spotify_show_url` (from a personal Save to Spotify) is **not** an RSS feed and must **not** trigger `--publish`; publishing is public and requires explicit consent.

**默认：只生成，不发布。**只有当用户明确要求发布、或播客目录中已有 RSS 订阅源——即带非空 `feed_url` 的 `"feed"` 对象（说明他们此前发布过订阅源、希望追加新节目）——时才加 `--publish`。只含 `spotify_show_url` 的 `"feed"` 对象（来自个人的 Save to Spotify）**不是** RSS 订阅源，**不得**触发 `--publish`；发布是公开行为，需要明确同意。

【评论】该条款把"个人保存到 Spotify 的节目链接"明确排除在 RSS 订阅源判定之外，避免把私密保存误当作公开发布的授权。

Always pass `--cover-prompt` with a short description of the episode's theme so podcast-helper can generate cover art automatically.

始终传入 `--cover-prompt` 并附一句对本期主题的简短描述，让 podcast-helper 能自动生成封面。

**Do not pass `--cover-image`.** Publishing accepts only generated cover art or the bundled default, so an episode published with `--cover-image` fails. Use `--cover-prompt` instead; if the user hands you an image file, generate a cover from a prompt describing it and say you did. The flag still parses and will be reconnected later — it just cannot reach the feed today.

**不要传 `--cover-image`。**发布只接受生成的封面或内置默认封面，因此带 `--cover-image` 发布的节目会失败。改用 `--cover-prompt`；如果用户递给你一个图片文件，就用一段描述它的提示词生成封面，并说明你这样做了。该标志仍可解析，以后会重新接上——只是今天到不了订阅源。

**Language:** unless the user asks for another language, write the title, description, feed copy, and script in the language of the current conversation. `podcast-helper` defaults synthesis to the request's `JARVIS_PRESENTATION_LOCALE`; pass `--language` only to honor an explicit user choice (e.g. `--language es`, `--language pt`, or a locale like `pt_BR`). Quality varies significantly across languages — English is highest quality; other languages can be rough (mispronunciations, accent), and some voices handle a given language better than others (try a different `--speaker` voice from `voice_source.json` if one sounds wrong). Tell the user non-English audio may be imperfect. See the `tts` skill's Language section for details.

**语言：**除非用户要求其他语言，标题、描述、订阅源文案与脚本都用当前对话的语言书写。`podcast-helper` 默认按请求的 `JARVIS_PRESENTATION_LOCALE` 合成；只有为尊重用户明确选择时才传 `--language`（如 `--language es`、`--language pt`，或 `pt_BR` 这样的区域设置）。各语言的合成质量差异明显——英语质量最高；其他语言可能比较粗糙（发音错误、口音），且不同嗓音对某一语言的表现有好有坏（若某个嗓音听起来不对，就从 `voice_source.json` 换一个 `--speaker` 嗓音）。要告诉用户非英语音频可能不完美。详见 `tts` 技能的 Language 一节。

**One-off episode:**  
**一次性节目：**  
```sh
podcast-helper generate \
  --script /tmp/script.txt \
  --title "Deep Dive into Async Rust" \
  --description "Alex and Jordan explore async patterns in Rust" \
  --speaker Alex=avocado_v2:MAI_03 \
  --speaker Jordan=avocado_v2:MAI_01 \
  --cover-prompt "async Rust programming, gears and lightning bolts" \
  --topics-file /tmp/topics.md
```

**Cron/recurring episode** — pass `--series-id` matching the cron job ID so episodes in the same series share cover art:  
**cron/常设节目**——传入与 cron 任务 ID 一致的 `--series-id`，让同一系列的节目共用封面：  
```sh
podcast-helper generate \
  --script /tmp/script.txt \
  --title "Morning News - June 2, 2026" \
  --description "Today's top stories" \
  --speaker Alex=avocado_v2:MAI_03 \
  --speaker Jordan=avocado_v2:MAI_01 \
  --series-id daily-news \
  --cover-prompt "morning news briefing, sunrise and newspaper" \
  --topics-file /tmp/topics.md
```

With `--series-id`, cover art is generated once for the first episode and reused for all subsequent episodes in the series.

带 `--series-id` 时，封面只为第一期生成一次，之后该系列的所有节目都复用它。

If the user asked to publish, or the podcast catalog already has an RSS feed — run `podcast-helper manifest read` and check for a `"feed"` object with a non-empty `feed_url` (a `feed` with only `spotify_show_url` does not count):

如果用户要求发布，或播客目录中已有 RSS 订阅源——运行 `podcast-helper manifest read`，检查是否存在带非空 `feed_url` 的 `"feed"` 对象（只含 `spotify_show_url` 的 `feed` 不算）：

```sh
podcast-helper generate \
  --script /tmp/script.txt \
  --title "Deep Dive into Async Rust" \
  --description "Alex and Jordan explore async patterns in Rust" \
  --speaker Alex=avocado_v2:MAI_03 \
  --speaker Jordan=avocado_v2:MAI_01 \
  --cover-prompt "async Rust programming" \
  --topics-file /tmp/topics.md \
  --publish --feed-title "Daily with Alex"
```

Returns JSON with `path`, `duration_secs`, `slug`, `cover`, and (if published) `feed_url`, `episode_url`, and `subscription_links`.

返回 JSON，含 `path`、`duration_secs`、`slug`、`cover`，以及（若已发布）`feed_url`、`episode_url` 和 `subscription_links`。

### 4. Deliver to chat / 在对话中交付
Always present the title as plain text above the playable link:  
始终把标题以纯文本形式放在可播放链接上方：  
```
{title}
[{title}](sandbox://workspace/podcasts/{slug}/{slug}.mp3)
```

The first time you mention publishing, a personal feed, or subscription in a conversation, briefly explain what the feed is: a personal RSS podcast feed they can add to a podcast app, and future published episodes will show up there automatically. After that first explanation, use shorter wording.

在对话中第一次提到发布、个人订阅源或订阅时，简要解释订阅源是什么：一个可加入播客客户端的个人 RSS 播客订阅源，今后发布的节目会自动出现在那里。第一次解释之后，改用更简短的说法。

If the episode was published, mention it was published and that it will appear in their feed. Note that published episodes are public. If the result has `published: false` with `publish_error`, the audio was generated locally but publication failed: relay `publish_error` to the user and do not claim it reached the feed. If it also has `blocked: true`, apply the content-review stop rule in step 5 and skip the publish follow-up below: do not retry, reword the script, or use another publish path. If it has `retryable: true`, content review could not run; tell the user publication can be retried later using the full step 5 `podcast-helper publish` command. Before retrying, run `podcast-helper manifest read` and reuse the target feed's existing `feed_title` exactly when one exists; never invent or alter a title for an existing feed.

如果节目已发布，说明已发布并会出现在其订阅源中。注意：已发布的节目是公开的。如果结果带 `published: false` 和 `publish_error`，说明音频已在本地生成但发布失败：把 `publish_error` 如实转达给用户，不要声称它已到达订阅源。如果还带 `blocked: true`，则执行第 5 步的内容审查停止规则，并跳过下文的发布后续追问：不要重试、不要改写脚本、也不要走其他发布途径。如果带 `retryable: true`，说明内容审查未能运行；告诉用户稍后可用第 5 步完整的 `podcast-helper publish` 命令重试发布。重试前，先运行 `podcast-helper manifest read`，若目标订阅源已有 `feed_title`，则原样复用；绝不为已存在的订阅源杜撰或更改标题。

After delivering, present applicable follow-ups:
交付之后，提出适用的后续选项：
1. **If not published and not blocked:** "Want me to publish this to a personal podcast feed? It's an RSS feed you can add to Apple Podcasts, Overcast, Pocket Casts, or most other podcast players, and future published episodes will show up there automatically. Note: published episodes are public — anyone with the link can listen."
   **若未发布且未被拦截：**"Want me to publish this to a personal podcast feed? It's an RSS feed you can add to Apple Podcasts, Overcast, Pocket Casts, or most other podcast players, and future published episodes will show up there automatically. Note: published episodes are public — anyone with the link can listen."
2. **If no cron job exists:** "Generate a new episode every day?"
   **若尚无 cron 任务：**"Generate a new episode every day?"

### 5. Publish (if not done in step 3) / 发布（若第 3 步未做）

```sh
podcast-helper publish \
  --slug {slug} \
  --feed-title "Daily with Alex" \
  --feed-description "Daily news and tech updates"
```

Returns JSON with `feed_url`, `episode_url`, and `subscription_links`.

返回 JSON，含 `feed_url`、`episode_url` 和 `subscription_links`。

Public publishing is content-reviewed. If the output has `"blocked": true`, the episode was **not** published: the audio and catalog entry still exist locally, but it cannot go to the public feed. Relay the returned `error` message to the user plainly and stop — do not retry, reword the script to get around it, or fall back to another publish path. Saving to the user's own Spotify show is unaffected by this review.

公开发布会经过内容审查。如果输出带 `"blocked": true`，该节目**未**发布：音频与目录条目仍在本地存在，但不能进入公开订阅源。把返回的 `error` 消息平实地转达给用户并停止——不要重试、不要为绕过审查而改写脚本、也不要退回其他发布途径。保存到用户自己的 Spotify show 不受此审查影响。

A publish that brings a new show cover unifies artwork across that show — its episodes' covers are replaced with it in the podcast catalog. A series show keeps its series cover, and an episode whose cover was changed with `update-cover` keeps the show's existing cover when published. Another feed's episodes keep their own artwork.

发布时若带来了新的节目封面，会把该节目的全部作品统一——目录中其各期节目的封面都被替换为它。系列节目保留其系列封面，而用 `update-cover` 改过封面的某期节目在发布时保留节目既有的封面。其他订阅源的节目保留各自的作品图。

After publishing, always re-present the listen link and confirm publication:  
发布之后，始终重新出示收听链接并确认发布：  
```
{title}
[{title}](sandbox://workspace/podcasts/{slug}/{slug}.mp3)

Published to your personal feed, which is the RSS feed you can subscribe to in your podcast app so future published episodes show up there automatically.
Published episodes are public — anyone with the feed link can listen.
```

### 6. Subscribe (after publishing) / 订阅（发布之后）
On the first episode published to a feed, present subscription links from the output JSON:

在向某个订阅源发布第一期节目时，出示输出 JSON 中的订阅链接：

```
Subscribe to your personal feed:

RSS feed URL (copy and paste into any podcast player):
`{subscription_links.rss}`

Or open directly in a podcast app:
- [Open in Apple Podcasts]({subscription_links.apple_podcasts})
- [Open in Overcast]({subscription_links.overcast})
- [Open in Pocket Casts]({subscription_links.pocket_casts})

This is the RSS feed for your generated episodes, and once you add it to a podcast app, future published episodes will appear there automatically.
```


**Important — how to render the feed URL:** The RSS feed URL is for the user to **copy and paste**, never to click. Always present any published `https://` URL (feed URL, episode link) wrapped in backticks as code — either inline `` `https://…` `` or inside a fenced code block. Never emit it as a bare URL (a bare `https://…` auto-linkifies on native clients) and never as a `[label](https://…)` markdown link; either form may fail to open on native clients and invites a click instead of a copy. Do not offer direct episode download links. The only clickable links should be local `sandbox://` listen links and the podcast-app protocol links (`podcast://`, `overcast://`, `pktc://`).

**重要——订阅源 URL 的呈现方式：**RSS 订阅源 URL 是供用户**复制粘贴**的，绝不是供点击的。任何已发布的 `https://` URL（订阅源 URL、节目链接）一律用反引号包成代码呈现——要么行内 `` `https://…` ``，要么放进围栏代码块。绝不要以裸 URL 输出（裸 `https://…` 在原生客户端上会自动变成链接），也绝不要作为 `[label](https://…)` 的 markdown 链接；这两种形式在原生客户端上可能无法打开，且会诱导点击而非复制。不要提供节目直接下载链接。唯一可点击的应当是本地 `sandbox://` 收听链接和播客客户端协议链接（`podcast://`、`overcast://`、`pktc://`）。

【评论】把订阅链接强制为代码样式，是针对聊天客户端自动把裸 URL 转成可点击链接的适配设计，目的是让用户复制而不是误点。

On subsequent episodes to the same feed, skip — the user is already subscribed.

对同一订阅源之后的节目，跳过此步——用户已订阅。

## Feed Organization / 订阅源组织

**Always use a single feed** unless the user explicitly asks for a separate one. Before publishing, run `podcast-helper manifest read`. `feeds` lists every feed on this VM and `feed` is whichever was published to most recently; if either has a non-empty `feed_url`, reuse that feed's `feed_title` exactly.

**始终只用一个订阅源**，除非用户明确要求单独一个。发布前运行 `podcast-helper manifest read`。`feeds` 列出本 VM 上的每个订阅源，`feed` 是最近一次发布到的那个；若其中任一具有非空 `feed_url`，就逐字复用该订阅源的 `feed_title`。

When the user does keep more than one show, the title is what picks the feed: publishing with a `--feed-title` that matches an existing feed adds to it, and any other title creates a new one. So reuse a title character for character when adding to a show, and never reuse one for a different show.

当用户确实保留多个节目时，标题就是挑选订阅源的依据：用与既有订阅源匹配的 `--feed-title` 发布会追加到它，任何其他标题则创建新的订阅源。因此向某个节目追加时逐字符复用其标题，且绝不在另一个节目上复用同一标题。

On the first publish (no `feed_url` in the podcast catalog yet — a `feed` object that only carries a `spotify_show_url` still counts as no RSS feed), choose a personal feed title based on the user's Muse name — e.g. "Today with Alex", "News with Alex".

首次发布时（播客目录中尚无 `feed_url`——只带 `spotify_show_url` 的 `feed` 对象仍视为没有 RSS 订阅源），基于用户的 Muse 名字选一个个人订阅源标题——如 "Today with Alex"、"News with Alex"。

## Add to your Spotify (personal, optional) / 添加到你的 Spotify（个人的，可选）
When the user explicitly asks to put a generated episode on their Spotify, use
`podcast-helper save-to-spotify` to add it to the user's **own** Spotify account. This is a personal
save to the user's own Spotify library/show — it is NOT the public/subscribable RSS feed publishing
described above, and does not use `podcast-helper --publish`. Only do this on an explicit request; do
not offer it proactively.

当用户明确要求把生成的某期节目放到其 Spotify 上时，使用 `podcast-helper save-to-spotify` 把它加入用户**自己的** Spotify 账户。这是对用户自己的 Spotify 资料库/show 的个人保存——它不是上文所述的公开/可订阅的 RSS 订阅源发布，也不使用 `podcast-helper --publish`。只在明确请求时执行；不要主动提议。

Run the upload through `podcast-helper` (not the raw `save-to-spotify` CLI): the helper uploads via the
bundled `save-to-spotify` CLI **and** records the resulting Spotify show link into the podcast catalog so
it shows up on the library podcasts page. See `references/save-to-spotify.md` for the underlying CLI,
the connect flow, and rules.

通过 `podcast-helper` 执行上传（而不是直接用 `save-to-spotify` CLI）：该助手经内置的 `save-to-spotify` CLI 上传，**并**把生成的 Spotify show 链接记入播客目录，使其显示在资料库播客页面上。底层 CLI、连接流程与规则见 `references/save-to-spotify.md`。

1. **List shows / check connection:** run `save-to-spotify --json shows` — it lists existing shows and
   confirms Spotify is connected. If it fails with a token / "not connected" error, tell the user to
   connect Spotify in **Settings → Connections → Spotify** (the shared Spotify connection), then retry.
   This tool has no `auth` subcommands or connect link. Ask the user whether to reuse an existing show
   or create a new one; don't silently pick.
   **列出 show / 检查连接：**运行 `save-to-spotify --json shows`——它列出现有 show 并确认 Spotify 已连接。若因 token / "not connected" 错误失败，告诉用户在 **Settings → Connections → Spotify**（共享的 Spotify 连接）中连接 Spotify，然后重试。此工具没有 `auth` 子命令或连接链接。询问用户是复用既有 show 还是新建一个；不要擅自选择。
2. **Upload + record** with `podcast-helper` (confirm title, target show, and summary with the user
   first — this writes to their account). The helper reads the audio/title/cover from the podcast catalog by
   slug; pass a cover only to override (must be JPEG/PNG ≤ 1 MB — convert `.webp`/other formats first):  
   **上传 + 记录**用 `podcast-helper`（先与用户确认标题、目标 show 与简介——这会写入他们的账户）。助手按 slug 从播客目录读取音频/标题/封面；只有要覆盖时才传封面（必须为 JPEG/PNG 且 ≤ 1 MB——先把 `.webp` 等其他格式转换）：  
   ```sh
   podcast-helper save-to-spotify \
     --slug {slug} \
     [--show-id <id> | --new-show "<show title>"] \
     [--title "{title}"] [--summary "{description}"] [--image {cover}]
   ```
   Returns JSON with `episode_id`, `episode_uri`, `spotify_show_id`, and `spotify_show_url`.
   返回 JSON，含 `episode_id`、`episode_uri`、`spotify_show_id` 和 `spotify_show_url`。
3. **Wait for readiness:** `save-to-spotify --json episodes status <episode-id> --wait` (use the
   `episode_id` from step 2). Processing is server-side and takes a few minutes; a returned
   `episode_uri` means it was accepted. If it stalls in `NOT_READY` (Spotify occasionally 503s), it's a
   Spotify-side delay — tell the user it's processing and retry later rather than polling indefinitely.
   **等待就绪：**`save-to-spotify --json episodes status <episode-id> --wait`（使用第 2 步的 `episode_id`）。处理在服务端进行，需要几分钟；返回 `episode_uri` 表示已被接受。若卡在 `NOT_READY`（Spotify 偶尔返回 503），那是 Spotify 侧的延迟——告诉用户正在处理、稍后重试，而不是无限轮询。
4. Tell the user it's on their Spotify and may take a few minutes to appear in the app. Because the show
   link is now recorded, the Spotify option on the podcasts library page will link to their show. Refer
   to the show and episodes by title and summarize readiness in plain language. Spotify show IDs,
   episode IDs, and `spotify:show:` / `spotify:episode:` URIs are internal CLI handles: use them for
   subsequent commands, but never include them in a user-facing response.
   告诉用户它已在用户的 Spotify 上，可能需要几分钟才会出现在客户端中。由于 show 链接已被记录，播客资料库页面上的 Spotify 选项将链接到他们的 show。用标题指称 show 与各期节目，并用平实语言概述就绪状态。Spotify show ID、episode ID 以及 `spotify:show:` / `spotify:episode:` URI 是内部 CLI 句柄：后续命令中使用它们，但绝不出现在面向用户的回复中。

For deletion, use the raw `save-to-spotify` CLI, not `podcast-helper`, `spotify-api`,
`podcasters.spotify.com`, or `creators.spotify.com`. Start with `save-to-spotify --json shows`, resolve
the requested show by title, and inspect it with `shows get <show-id>`. Use
`episodes --show-id <show-id>` followed by `episodes delete <episode-id>` to remove one episode. To
remove the entire show, confirm that all its episodes will be deleted and run `shows delete <show-id>`;
that command removes the episodes too. Treat `{"status":"deleted"}` as accepted, then poll the relevant
`shows` or `episodes --show-id` inventory for up to 60 seconds. Confirm deletion only after the item is
absent; otherwise tell the user Spotify is still propagating the accepted deletion and do not repeat it.
Deleting the Spotify copy does not delete local audio or an RSS feed. See
`references/save-to-spotify.md` for the full command flow.

删除时使用原始的 `save-to-spotify` CLI，而不是 `podcast-helper`、`spotify-api`、`podcasters.spotify.com` 或 `creators.spotify.com`。先运行 `save-to-spotify --json shows`，按标题解析所请求的 show，并用 `shows get <show-id>` 查看。用 `episodes --show-id <show-id>` 加 `episodes delete <episode-id>` 删除单期节目。要删除整个 show，先确认其所有节目都将被删除，然后运行 `shows delete <show-id>`；该命令会一并删除各期节目。把 `{"status":"deleted"}` 视为已受理，然后对相关的 `shows` 或 `episodes --show-id` 清单轮询至多 60 秒。只有当条目确实消失后才确认删除；否则告诉用户 Spotify 仍在传播这条已受理的删除，并且不要重复执行。删除 Spotify 副本不会删除本地音频或 RSS 订阅源。完整命令流程见 `references/save-to-spotify.md`。

## Cover Art / 封面

Cover art is handled automatically by `podcast-helper` during generation. You control it with two flags:

封面在生成过程中由 `podcast-helper` 自动处理。你用两个标志控制它：

- **`--cover-prompt`** — a short description of the episode theme (e.g. "morning news briefing, sunrise and cityscape"). podcast-helper shells out to `media-generation` to create a square icon-style image. Always provide this.
  **`--cover-prompt`**——对节目主题的一句话描述（如 "morning news briefing, sunrise and cityscape"）。podcast-helper 会调用 `media-generation` 生成方形图标风格的图片。始终提供此项。
- **`--series-id`** — groups episodes that share the same cover art. Use the cron job ID for recurring episodes. Without this, each episode gets unique art.
  **`--series-id`**——把共用同一封面的节目归为一组。常设节目使用 cron 任务 ID。不传则每期节目获得独一无二的封面。

**How artwork flows:**

**作品图的流转方式：**

| Scenario | Behavior |
|----------|----------|
| One-off episode | Unique cover generated per episode from `--cover-prompt` |
| Recurring/cron episode | First episode generates cover; subsequent episodes with same `--series-id` reuse it |
| Published episode | A new show cover replaces that feed's episode covers; a series show, or an episode changed with `update-cover`, keeps the show's cover |

| 场景 | 行为 |
|----------|----------|
| 一次性节目 | 每期由 `--cover-prompt` 生成独一无二的封面 |
| 常设/cron 节目 | 第一期生成封面；之后相同 `--series-id` 的各期复用它 |
| 已发布节目 | 新的节目封面会替换该订阅源各期节目的封面；系列节目、或用 `update-cover` 改过封面的某期，发布时保留节目既有封面 |

**Fallback** when no `--cover-prompt` is provided: the bundled default cover
(`/opt/hatch/skills/generate_podcast/default-cover.jpg`). The user's avatar is
also tried first, but an avatar cover cannot currently be published — one more
reason to always pass `--cover-prompt` on an episode headed for a feed.

未提供 `--cover-prompt` 时的**兜底**：内置默认封面（`/opt/hatch/skills/generate_podcast/default-cover.jpg`）。也会先尝试用户头像，但头像封面目前无法发布——这是面向订阅源的节目始终要传 `--cover-prompt` 的又一个理由。

You do NOT need to call `media.generate_image` yourself for cover art — `podcast-helper` handles it internally.

你不需要为封面亲自调用 `media.generate_image`——`podcast-helper` 内部处理。

`--cover-image` is not available right now: publishing takes only generated cover art or the bundled default. The flag is still accepted so it can be reconnected later, but an episode published with it fails, so do not use it.

`--cover-image` 目前不可用：发布只接受生成的封面或内置默认封面。该标志仍被接受以便日后重新接上，但带它发布的节目会失败，所以不要使用。

### Changing the cover / 更换封面

When the user asks to change an episode's cover — a new prompt, a variation, or an image they hand you — regenerate through the helper so the catalog reference moves to a new file:

当用户要求更换某期节目的封面——新提示词、变体、或他们递来的图片——通过助手重新生成，让目录引用指向新文件：

```sh
podcast-helper update-cover \
  --slug {slug} \
  --cover-prompt "starry night sky over a mountain lake"
```

This generates new artwork to a fresh path, updates the episode's `cover` reference to it, and prints the new path as `cover`. Present that new path in chat.

它会生成新的作品图到一个全新路径，把该期节目的 `cover` 引用更新为它，并把新路径作为 `cover` 打印出来。在对话中出示这个新路径。

Hard rules:

硬性规则：

- **Never overwrite the current cover file.** Do not `cp`, move, or rename a new image over the old cover path, do not call `media.generate_image` for covers, and do not present the old path again after changing its bytes. Already-delivered previews and library rows reference the old file by path, and clients cache by path — rewriting its bytes silently changes what was already shown (or shows stale art), which is the failure this flow exists to prevent.
  **绝不覆盖当前封面文件。**不要把新图片 `cp`、移动或重命名到旧封面路径上，不要为封面调用 `media.generate_image`，也不要在更改其字节后再次出示旧路径。已交付的预览与资料库条目按路径引用旧文件，客户端按路径缓存——改写字节会悄悄改变已展示的内容（或显示陈旧的图），这正是这条流程要防止的失败。
- **A user-supplied image is a prompt, not a file.** Describe it in `--cover-prompt` and generate (the same rule as `--cover-image` at generation time); publishing still takes only generated art.
  **用户提供的图片是提示词，不是文件。**在 `--cover-prompt` 中描述它并生成（与生成时的 `--cover-image` 同一规则）；发布仍只接受生成的作品图。
- **Series episodes diverge.** `update-cover` changes only that episode in the local catalog; the show cover, the other episodes, and future `--series-id` generations keep the shared series cover. Publishing the edited episode keeps the show's cover too, and the public feed shows that cover (a feed has one cover).
  **系列节目各自分叉。**`update-cover` 只在本地目录中更改该期节目；节目封面、其他各期、以及未来带 `--series-id` 的生成仍保留共享的系列封面。发布被编辑的那期也保留节目的封面，公开订阅源显示的也是那个封面（一个订阅源只有一个封面）。
- **Published feeds keep the previous art, and publish refuses repeats.** `podcast-helper publish` refuses an already-published episode, because publishing appends a NEW episode entry (duplicating the episode in every subscriber's feed) while the catalog row forgets the original. So there is no publish path that pushes a cover change: a cover-only refresh does not exist yet, and cover edits on published shows stay local to the catalog and chat.
  **已发布订阅源保留旧图，发布拒绝重复。**`podcast-helper publish` 拒绝已发布的节目，因为发布会追加一条新的节目条目（在每个订阅者的订阅源中造成该节目重复），而目录行会遗忘原始条目。因此不存在能推送封面更改的发布途径：目前没有仅刷新封面的功能，已发布节目上的封面编辑只留在目录与对话的本地层面。

【评论】"封面写入新路径而非覆盖旧文件"是围绕客户端按路径缓存图片这一现实约束的设计；同理，发布拒绝重复条目是为了避免订阅源中出现重复节目。

## Scheduling / 排期
When a user asks for a recurring podcast, audio briefing, or scheduled audio content:

当用户要求常设播客、音频简报或定时音频内容时：

1. **Generate a first episode now** — don't just set up the cron and leave them with nothing to listen to. Generate and deliver the first episode immediately so the user has something right away.
   **立即生成第一期**——不要只搭好 cron 就让用户无节目可听。立刻生成并交付第一期，让用户马上有东西可听。
2. **Ask about publishing** — offer to publish to a podcast feed so they can subscribe and listen in their preferred podcast app. If they agree, publish the first episode and present subscription links.
   **询问是否发布**——提议发布到播客订阅源，让他们能订阅并在自己喜欢的播客客户端中收听。若同意，发布第一期并出示订阅链接。
3. **Then create the cron job** — set up the recurring schedule.
   **然后再创建 cron 任务**——设置常设排期。

For recurring podcasts, create a cron job using the `cron` tool. **Always set `timeout_secs: 1800`**.

对常设播客，用 `cron` 工具创建 cron 任务。**始终设置 `timeout_secs: 1800`**。

Example `cron` tool call:  
`cron` 工具调用示例：  
```json
{
  "action": "add",
  "file_name": "daily-podcast__daily@13:00:00.md",
  "id": "daily-podcast",
  "enabled": true,
  "mode": "task",
  "timeout_secs": 1800,
  "schedule": {
    "kind": "daily",
    "timezone": "UTC",
    "time": "13:00:00"
  },
  "body": "Generate a new podcast episode on <topic>. Follow the generate_podcast skill workflow. Keep the same cast every episode: hosts Alex (--speaker Alex=avocado_v2:MAI_01) and Jordan (--speaker Jordan=avocado_v2:MAI_03). Use --series-id daily-podcast and --cover-prompt '<topic description>' when calling podcast-helper generate. Before choosing today's angle, run podcast-helper manifest read and review each recent episode's topics field to avoid repeating what was already covered. Write a short topics summary to /tmp/topics.md and pass it with --topics-file so future episodes can dedup against it. Publish to the feed after generating."
}
```

Keep cron task descriptions concrete — include the topic angle, the fixed cast (host names and their voice ids, so every episode uses the same hosts and voices), `--series-id` matching the cron ID, and explicit instructions to research live results at generation time.

保持 cron 任务描述具体——包含主题角度、固定班底（主持人姓名与其嗓音 id，让每期用相同的主持人与嗓音）、与 cron ID 匹配的 `--series-id`，以及"在生成时检索实时结果"的明确指示。

## Utility Commands / 实用命令

For edge cases and manual operations, `podcast-helper` exposes sub-commands:

针对边界情况与手动操作，`podcast-helper` 暴露以下子命令：

- `podcast-helper manifest read` — print the current podcast catalog
  `podcast-helper manifest read`——打印当前播客目录
- `podcast-helper manifest add-episode --slug ... --title ... --duration-secs ... --path ... --chunk-count ...` — add episode
  `podcast-helper manifest add-episode --slug ... --title ... --duration-secs ... --path ... --chunk-count ...`——添加节目
- `podcast-helper manifest update-episode --slug ... [--episode-url ...] [--feed-url ...]` — update episode
  `podcast-helper manifest update-episode --slug ... [--episode-url ...] [--feed-url ...]`——更新节目
- `podcast-helper update-cover --slug ... --cover-prompt ...` — replace an episode's cover with newly generated art (updates the reference; never overwrites the old file)
  `podcast-helper update-cover --slug ... --cover-prompt ...`——用新生成的作品图替换某期节目的封面（更新引用；绝不覆盖旧文件）
- `podcast-helper generate-slug --title "..."` — generate a kebab-case slug with today's date
  `podcast-helper generate-slug --title "..."`——生成带今日日期的 kebab-case slug

## Listening to Existing Episodes / 收听既有节目
If the user asks to listen to a podcast, hear their podcast, or asks about their episodes, read the podcast catalog with `podcast-helper manifest read` and present listen links for the relevant episodes:

如果用户要求收听某个播客、听听他们的播客，或询问其各期节目，用 `podcast-helper manifest read` 读取播客目录，并出示相关节目的收听链接：

`[{title}](sandbox://workspace/podcasts/{slug}/{slug}.mp3)`

If the feed is published, also include the subscription links (see section 6 format).

若订阅源已发布，同时附上订阅链接（格式见第 6 节）。

## Operating Rules / 操作规则
1. Use voices from `/opt/hatch/skills/voice-selector/voice_source.json`, referenced by their catalog `id`. Default to the Meta AI voices (catalog ids `MAI_01` and `MAI_03`) most of the time, but pick another catalog voice when the topic, tone, or user request steers that way. Never say "MAI" to the user — call them the Meta AI voices or use display names.
   使用 `/opt/hatch/skills/voice-selector/voice_source.json` 中的嗓音，按其目录 `id` 引用。多数情况下默认使用 Meta AI 嗓音（目录 id `MAI_01` 与 `MAI_03`），但当主题、语气或用户请求指向别处时，可挑选目录中的其他嗓音。绝不对用户说 "MAI"——称之为 Meta AI voices 或使用展示名。
2. If a chunk fails, `tts synthesize-script` stops — do not deliver a partial episode. Most failures are transient backend issues: **retry the same generation later** on a bounded backoff (~5m, ~10m, ~30m, ~1h), keeping the **same speaker voices**. Never swap in a different voice or a different TTS engine to work around a failure. Schedule the retry (a delayed wakeup or short cron) rather than blocking, and tell the user you'll deliver the episode once synthesis recovers. **If it still fails after the ~1h retry, stop** — cancel the scheduled retry, report the error, and suggest they try again later. (A `... not allowed to use voiceID ...` error won't clear on retry — that voice id isn't permitted for this client; pick another from `voice_source.json` and regenerate. Auth or clear request errors likewise need a fix, not a retry.)
   若某个分块失败，`tts synthesize-script` 会停止——不要交付残缺的节目。多数失败是暂时性的后端问题：**稍后重试同一生成**，按有界的退避（约 5 分钟、10 分钟、30 分钟、1 小时），并保持**相同的说话人嗓音**。绝不为绕过失败而换用别的嗓音或别的 TTS 引擎。安排重试（延迟唤醒或短 cron）而不是阻塞等待，并告诉用户合成恢复后会交付节目。**若约 1 小时重试后仍失败，停止**——取消已安排的重试，报告错误，并建议用户稍后再试。（`... not allowed to use voiceID ...` 错误重试不会消除——该嗓音 id 未获此客户端许可；从 `voice_source.json` 换一个并重新生成。鉴权错误或明显的请求错误同样需要修复，而不是重试。）
3. On first episode in a new feed, help the user subscribe. On subsequent episodes, skip.
   新订阅源的第一期时，协助用户订阅。之后的节目跳过。
4. **Cron runs:** Follow whatever the cron task description says. If it says to publish, publish without confirmation.
   **cron 运行：**按 cron 任务描述执行。若描述要求发布，无需确认直接发布。
5. **Never** present a published `https://` feed or episode URL as a bare or clickable link — always as copyable code (wrapped in backticks). Only `sandbox://` listen links and podcast-app protocol links (`podcast://`, `overcast://`, `pktc://`) may be clickable.
   **绝不**把已发布的 `https://` 订阅源或节目 URL 以裸链接或可点击链接呈现——一律作为可复制的代码（用反引号包裹）。只有 `sandbox://` 收听链接和播客客户端协议链接（`podcast://`、`overcast://`、`pktc://`）可以点击。
