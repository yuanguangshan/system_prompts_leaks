---
name: magic-moment
metadata: { "includeInPrompt": false }
description: Make a "magic moment" video — turn a creator's talking-head recording into a vertical video preserving the source narration where their Muse story replays through artifacts and brief exchanges synced to their voiceover — bubbles, typing, emoji reactions, real widgets and pages, message sounds, closing Muse lockup finisher. Use whenever a user with talking-head or selfie footage wants it turned into a shareable clip of their Muse story — "make a magic moment", "turn this video of me into...", "add the chat over my video", "retell what Muse did for me" — even if they never say the words "magic moment".
---

<!-- BILINGUAL-EN-ZH -->
# Magic Moment / 魔法时刻

The build is a guided pipeline; read the guides in order and follow them
exactly — this file is only the map.

构建是一条有引导的流水线；按顺序阅读各指南并严格遵循——本文件只是地图。

1. `guide/story_and_canon.md` — transcribe the clip, ground every beat in
   the real work this VM actually did, and lock the fact sheet.
   `guide/story_and_canon.md` — 转录片段，把每个节拍都锚定在本 VM 真正做过的实际工作上，并锁定事实清单。
2. `guide/conversation_shape.md` — choose the exchanges and artifact reveals that carry the story.
   `guide/conversation_shape.md` — 选择承载故事的对话往来与工件揭晓。
3. `guide/visuals.md` — plan and preview the visuals. Match the component
   to the narrated action and give the actual artifact room to be seen.
   `guide/visuals.md` — 规划并预览视觉。把组件与旁白所述动作匹配，并给真实工件留出被看清的空间。
4. `guide/screenplay.md` and `guide/timeline.md` — assemble beats.
   `guide/screenplay.md` 与 `guide/timeline.md` — 组装节拍。
5. `guide/screenplay_review.md` — the review gate before any render.
   `guide/screenplay_review.md` — 任何渲染之前的评审关卡。

The look is owned by `reference/design.md` (doctrine) and
`reference/design-system/muse-moments-kit.html` (product components).  
Use `reference/visual-storytelling.md` for diagram composition and motion. Card mechanics: `reference/card-spec.md`; renderer  
contracts: `reference/overlay-spec.md`.

视觉外观由 `reference/design.md`（准则）与
`reference/design-system/muse-moments-kit.html`（产品组件）负责。  
图解构图与动效使用 `reference/visual-storytelling.md`。卡片机制：`reference/card-spec.md`；渲染器契约：`reference/overlay-spec.md`。

Tools: `./mm transcribe | validate | preview | render | inspect | publish | webshots | snap | avatar |
example`. Run `./mm webshots` first on any browser story: it copies the
browser's own captures of the pages it drove. Read and crop from them
before you rebuild those pages for the browser beat. `./mm snap`
captures artifacts and live pages.

工具：`./mm transcribe | validate | preview | render | inspect | publish | webshots | snap | avatar | example`。任何浏览器故事都先运行 `./mm webshots`：它会复制浏览器对自己所驱动页面的截图。在为浏览器节拍重建那些页面之前，先读取并裁剪这些截图。`./mm snap` 捕获工件与实时页面。

## Component index / 组件索引

This index names every component in the kit so you can find one by capability.
Grep the label in  
`/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html`  
and copy that component's markup, which is the source of truth for the look;
`/opt/hatch/skills/magic-moment/reference/design.md` owns how to copy, scale,
and adapt it. The kit's foundations (palette, type, rules) and its
thread-chrome section are renderer-drawn reference, not components to pick.

本索引命名套件中的每个组件，便于按能力查找。在  
`/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html`  
中 grep 该标签，并复制该组件的标记，它是外观的唯一事实来源；`/opt/hatch/skills/magic-moment/reference/design.md` 负责如何复制、缩放与改写。套件的基础（调色板、字体、规则）与 thread-chrome 部分是渲染器绘制的参考，不是可挑选的组件。

【评论】front matter 的 description 罗列多种同义触发语并写明"即使用户不说 magic moment 也要触发"，属于刻意扩大触发面的写法；而 `includeInPrompt: false` 表示正文不进入系统提示词。

- Artifacts & documents / 工件与文档
  - `web artifact · finished (hero)`: a page Muse built, as its real screenshot.
    `web artifact · finished (hero)`：Muse 构建的页面，以其真实截图呈现。
  - `fullstack app · resting (icon card, figma space icon)`: the app Muse built, at rest before the tap.
    `fullstack app · resting (icon card, figma space icon)`：Muse 构建的应用，点击前静置的状态。
  - `fullstack app · tapped (a legit full web app)`: the tap beat that opens that app full screen.
    `fullstack app · tapped (a legit full web app)`：点击节拍，全屏打开该应用。
  - `file card · shipping widget replica`: a document Muse produced, with the real file in the preview well.
    `file card · shipping widget replica`：Muse 生成的文档，真实文件放在预览井中。
  - `slides card · shipping widget replica`: a slide deck Muse produced.
    `slides card · shipping widget replica`：Muse 生成的幻灯片组。
  - `user upload chip · shipping replica`: a file the user uploaded, on the user side of the thread.
    `user upload chip · shipping replica`：用户上传的文件，位于会话的用户一侧。
  - `completion card`: work finished when no other component covers what Muse did.
    `completion card`：当没有其他组件能覆盖 Muse 所做的工作时，表示工作完成。
  - `external link · shipping widget replica`: a link Muse sent, unfurled with its hero and host.
    `external link · shipping widget replica`：Muse 发送的链接，展开显示其主图与来源站点。
- Media / 媒体
  - `generated image · renders as-is`: an image Muse generated, on its own image beat.
    `generated image · renders as-is`：Muse 生成的图像，放在它自己的图像节拍上。
  - `generated video · renders as-is`: a video Muse generated, on its own video beat.
    `generated video · renders as-is`：Muse 生成的视频，放在它自己的视频节拍上。
  - `image picker · mobile canonical (2×2 + quote-reply)`: the user picking one of several generated images.
    `image picker · mobile canonical (2×2 + quote-reply)`：用户从多张生成图像中挑选一张。
  - `music · found and playing`: a track or mix Muse found or made, playing.
    `music · found and playing`：Muse 找到或制作的曲目/混音，正在播放。
  - `media library · organized`: photos or media Muse sorted into groups.
    `media library · organized`：Muse 分组整理的照片或媒体。
- Browser & shopping / 浏览器与购物
  - `browser · rebuilt journey (one card, tap by default)`: a site Muse drove; author the browser beat's pages rather than copying this markup.
    `browser · rebuilt journey (one card, tap by default)`：Muse 驱动过的网站；浏览器节拍的页面要自行编写，而不是复制此标记。
  - `skill task · status chip (no browser involved)`: a skill or connector did the work instead of the browser.
    `skill task · status chip (no browser involved)`：由技能或连接器而非浏览器完成的工作。
  - `web search · citations`: an answer Muse sourced from the web, with its citations.
    `web search · citations`：Muse 从网络取材并附引用的回答。
  - `shopping results · shipping widget replica`: products Muse found, as the in-bubble result grid.
    `shopping results · shipping widget replica`：Muse 找到的商品，以气泡内的结果网格呈现。
  - `purchase · order placed (canonical figma)`: an order Muse placed.
    `purchase · order placed (canonical figma)`：Muse 下的一笔订单。
  - `browser · activity island (canonical figma)`: browser work in flight, as the floating glass island.
    `browser · activity island (canonical figma)`：进行中的浏览器工作，以悬浮玻璃岛呈现。
- Trust & security / 信任与安全
  - `connect an account · in-chat link + disclosure sheet (canonical figma)`: the user connecting an account Muse needs.
    `connect an account · in-chat link + disclosure sheet (canonical figma)`：用户连接 Muse 所需的账户。
  - `secure storage · log in details (figma canonical)`: the log-ins already held in secure storage.
    `secure storage · log in details (figma canonical)`：已保存在安全存储中的登录信息。
  - `secure storage · add log in details (figma canonical)`: saving a new log-in to secure storage.
    `secure storage · add log in details (figma canonical)`：把新登录信息保存进安全存储。
  - `sentinel · approval card (figma canonical)`: Muse asking permission before a sensitive action.
    `sentinel · approval card (figma canonical)`：Muse 在敏感操作前请求许可。
  - `secure form fill`: Muse filling a card number or password from secure storage.
    `secure form fill`：Muse 从安全存储填入卡号或密码。
- Communication / 通信
  - `texting on your behalf · message thread (send animates)`: a text Muse sent for the user, in a real message thread.
    `texting on your behalf · message thread (send animates)`：Muse 替用户发送的短信，置于真实短信会话中。
  - `email · triage + draft ready`: an inbox Muse triaged, with a draft waiting.
    `email · triage + draft ready`：Muse 分拣过的收件箱，一封草稿待发。
  - `email · draft in the real gmail composer (send animates)`: an email Muse drafted and sent from the Gmail composer.
    `email · draft in the real gmail composer (send animates)`：Muse 在 Gmail 撰写器中起草并发送的邮件。
  - `phone call · live + outcome`: a call Muse placed, live and then its result.
    `phone call · live + outcome`：Muse 拨打的电话，先实时通话再显示结果。
  - `audio pill · shipping widget replica`: a voice memo or audio clip in the thread.
    `audio pill · shipping widget replica`：会话中的语音备忘或音频片段。
- Automation & ambient / 自动化与环境感知
  - `scheduled tasks · agent list (canonical figma)`: the tasks Muse runs on a schedule.
    `scheduled tasks · agent list (canonical figma)`：Muse 按计划运行的任务。
  - `standing watch · ambient guardian`: Muse watching something over days, such as a price.
    `standing watch · ambient guardian`：Muse 连续多日监视某个对象，例如价格。
  - `proactive nudge · text_with_button replica`: an unprompted nudge with one call to action.
    `proactive nudge · text_with_button replica`：带单一行动召唤的主动提示。
  - `option widget · dashed choices (shipping replica)`: the user picking one of several written choices.
    `option widget · dashed choices (shipping replica)`：用户从若干书面选项中挑一个。
  - `letter · shipping widget replica`: a letter Muse wrote.
    `letter · shipping widget replica`：Muse 写的一封信。
  - `idea card · shipping widget replica`: one idea Muse surfaced, with its preview and its pill. `From your calls` is this card's meta line, not a component of its own.
    `idea card · shipping widget replica`：Muse 呈现的一个点子，带其预览与胶囊标签。`From your calls` 是该卡的元信息行，不是独立组件。
  - `morning brief · feed edition (canonical figma)`: the morning feed edition Muse published. `Your day` and `Heads up` are section headers inside it, not components of their own.
    `morning brief · feed edition (canonical figma)`：Muse 发布的晨间信息流版。`Your day` 与 `Heads up` 是其中的栏目标题，不是独立组件。
  - `ideas · idea rows (canonical figma)`: the ideas list, with one row being chosen.
    `ideas · idea rows (canonical figma)`：点子列表，其中一行被选中。
- Personal intelligence / 个人智能
  - `memory · recalled from weeks ago`: Muse acting on something the user said weeks earlier.
    `memory · recalled from weeks ago`：Muse 依据用户几周前说过的话行动。
  - `memory · person page (markdown doc, product design)`: a person's memory page and its open threads.
    `memory · person page (markdown doc, product design)`：某人的记忆页及其未结对话串。
  - `goals · goals surface (canonical figma)`: the user's goals list, with a subgoal checking off.
    `goals · goals surface (canonical figma)`：用户的目标列表，一个子目标被勾选完成。
  - `goals · tracking check-in (canonical figma)`: a check-in on a habit Muse tracks.
    `goals · tracking check-in (canonical figma)`：Muse 追踪的一个习惯的打卡。
  - `calendar · conflict resolved`: a calendar conflict Muse moved.
    `calendar · conflict resolved`：Muse 移开的日历冲突。
  - `device sync · flowing in`: the user's device data syncing in.
    `device sync · flowing in`：用户的设备数据同步流入。
- Work & analysis / 工作与分析
  - `deep research · multi-source report`: a multi-source report Muse researched.
    `deep research · multi-source report`：Muse 研究的多来源报告。
  - `uploaded file · analyzed`: a document the user uploaded and Muse read.
    `uploaded file · analyzed`：用户上传且 Muse 读过的文档。
  - `data analysis · chart from a spreadsheet`: numbers Muse charted from a spreadsheet.
    `data analysis · chart from a spreadsheet`：Muse 从电子表格绘制的数字图表。
  - `long thread · digest`: a long thread Muse condensed into decisions.
    `long thread · digest`：Muse 浓缩为决策要点的一条长会话串。
- Meta-capabilities / 元能力
  - `subagents · fanning out in parallel`: work Muse ran across several helpers at once.
    `subagents · fanning out in parallel`：Muse 同时分派给多个助手并行执行的工作。
  - `wallet · muse wallet screen (canonical figma)`: the wallet screen and its payment methods.
    `wallet · muse wallet screen (canonical figma)`：钱包界面及其支付方式。
  - `wallet · purchase approval (canonical figma)`: the user approving a purchase Muse is about to make.
    `wallet · purchase approval (canonical figma)`：用户批准 Muse 即将进行的一笔购买。
  - `title card · chapter divider (canonical figma)`: a full-bleed chapter divider between story sections.
    `title card · chapter divider (canonical figma)`：故事章节之间的通栏分隔页。
