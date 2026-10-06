<!-- BILINGUAL-EN-ZH -->
# Choose visuals from the evidence / 从证据中选择视觉素材

Read the component index in `/opt/hatch/skills/magic-moment/SKILL.md` and the relevant component in `/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html`. Choose a component that depicts the supported action and state. Treat all gallery names, values, and sample copy as examples, never evidence. Reuse the real artifact when several narration beats describe different parts of it.

阅读 `/opt/hatch/skills/magic-moment/SKILL.md` 中的组件索引，以及 `/opt/hatch/skills/magic-moment/reference/design-system/muse-moments-kit.html` 中的相关组件。选择一个能够描绘所支持动作和状态的组件。把所有图库名称、取值和示例文案都当作示例，绝不当作证据。当多个旁白节拍描述同一真实工件的不同部分时，复用该真实工件。

Prefer original media and captures of actual artifacts. Record `provenance: {"kind": "original-media|historical-capture|current-capture|reconstructed-ui|synthetic-illustration", "source": "source path or event and displayed state"}` on each file-backed visual or video. A file path alone establishes neither authenticity nor historical state.

优先使用原始媒体和真实工件的捕获。在每个基于文件的视觉素材或视频上记录 `provenance: {"kind": "original-media|historical-capture|current-capture|reconstructed-ui|synthetic-illustration", "source": "source path or event and displayed state"}`。仅凭一个文件路径既不能证明真实性，也不能证明历史状态。

Keep provenance in the screenplay and review files. Do not print verification
labels such as "actual artifact," "historical copy," or test-VM status on the
video. Show the artifact's own name and content. Write visible copy as the product
would. Remove sentences explaining how the video was constructed or reviewed;
let the composition and concise state labels carry those distinctions. When timing matters to the
claim, use its relevant date or a concise state such as "Planned." Do not add
"Live," a sync-success badge, or a completion animation unless the source
supports that state. Omit an unsupported claim instead of covering it with a
technical disclaimer.

把来源信息（provenance）保存在剧本和审查文件中。不要在视频上印上"真实工件"、"历史副本"或测试虚拟机状态之类的验证标签。展示工件自身的名称和内容。可见文案要按产品本来的方式撰写。删除那些解释视频如何制作或审查的句子；让构图和简洁的状态标签来传达这些区别。当时机与论断相关时，使用相关日期或"已规划"（Planned）这样简洁的状态。除非来源支持该状态，不要添加"实时"（Live）、同步成功徽章或完成动画。宁可省略没有依据的论断，也不要用技术性免责声明来掩盖它。

【评论】来源信息只写入剧本与审查文件、不印到视频画面上，并禁止无依据的"实时/成功"状态展示，属于防止成品夸大宣传的设计。

Read `/opt/hatch/skills/magic-moment/reference/visual-storytelling.md` before
authoring. Write a short scene plan with each beat's visual focus, component,
source, and motion. Run
`/opt/hatch/skills/magic-moment/mm preview <screenplay.json>` and open every
card's generated preview before committing to a full render. Inspect each
card's initial, middle, and final animation states at its intended size in a
360px-wide video, including its expanded state
when tapped. Fix clipped text, crowded headings, and weak artifact reveals
before rendering again.

在创作之前先阅读 `/opt/hatch/skills/magic-moment/reference/visual-storytelling.md`。写一份简短的场景计划，包含每个节拍的视觉焦点、组件、来源和动效。运行 `/opt/hatch/skills/magic-moment/mm preview <screenplay.json>`，在决定完整渲染之前打开每张卡片的生成预览。在 360px 宽的视频中，按预期尺寸检查每张卡片的初始、中间和最终动画状态，包括点按后的展开状态。再次渲染之前，先修复被裁切的文字、拥挤的标题和乏味的工件展示。

When a source artifact is mostly prose, build a designed presentation of its
supported content with a clear focal result and readable supporting details.
Save that presentation as an artifact before capturing it. Preserve the source
facts and record the adaptation in provenance. Do not shrink a full page of
paragraphs into a card or invent data to make a chart look interesting.

当来源工件以大段文字为主时，为其受支持的内容构建一个经过设计的呈现，具有清晰的核心结果和可读的支撑细节。先把这个呈现保存为工件，然后再捕获它。保留来源事实，并在 provenance 中记录这一改编。不要把整页段落塞进一张卡片，也不要为了让图表显得有趣而编造数据。

Use `/opt/hatch/skills/magic-moment/mm webshots` to find browser captures. Match the source task, URL, and time to the narrated journey before using one. The presence of unrelated captures does not establish that this story used the browser. Use a browser beat only for an evidenced sequence of browser actions.

使用 `/opt/hatch/skills/magic-moment/mm webshots` 查找浏览器捕获。在使用之前，把来源任务、URL 和时间与旁白叙述的历程进行匹配。存在无关捕获并不能证明这个故事使用过浏览器。只有对有证据支持的浏览器操作序列才使用浏览器节拍。

Use `/opt/hatch/skills/magic-moment/mm snap <document-or-url> --run <name>` for a current capture. This opens a fresh browser context, without the product's cookies, local storage, or state bridge. Inspect the displayed state. Use the supported product browser route when an artifact needs live state; do not treat the default local HTML state as the user's data. When the fresh context shows empty counters or sample state, crop a complete supported block that excludes them or reconstruct the narrated state from saved records and record that provenance. Do not click a mutation control to populate a screenshot. Use `--viewport-height <pixels>` to capture an explicit viewport or capture a complete meaningful block when the page exceeds the size budget; the renderer rejects silent truncation.

使用 `/opt/hatch/skills/magic-moment/mm snap <document-or-url> --run <name>` 进行当前状态捕获。这会打开一个全新的浏览器上下文，不带产品的 cookie、本地存储或状态桥。检查所显示的状态。当工件需要实时状态时，使用受支持的产品浏览器路径；不要把默认的本地 HTML 状态当作用户的数据。当全新上下文显示空计数器或示例状态时，裁切出一个排除它们的完整受支持区块，或从已保存的记录重建旁白所述状态，并在 provenance 中记录。不要通过点击修改类控件来填充截图。用 `--viewport-height <pixels>` 捕获明确的视口，或在页面超出尺寸预算时捕获一个完整的有意义区块；渲染器拒绝静默截断。

【评论】禁止点击修改类控件来填充截图、不把本地示例状态当作用户数据，是对"演示画面必须与真实状态一致"的约束。

Read `/opt/hatch/skills/magic-moment/reference/card-spec.md` for cropping and `/opt/hatch/skills/magic-moment/reference/design.md` for the kit's design rules. Keep sourced content legible at phone scale. Use one content focus per card and remove unsupported fields. Use the kit's shipped Optimistic fonts. Keep authored cards self-contained at 1240px wide, with at least 32px visible type and a 56px primary line. Preserve meaningful inner structure under the one-surface rule in `/opt/hatch/skills/magic-moment/reference/design.md`. Use `tap: true` for a supported reveal that benefits from enlargement; ordinary cards have a 600px height budget and tap cards 1100px.

裁切规则见 `/opt/hatch/skills/magic-moment/reference/card-spec.md`，组件库的设计规则见 `/opt/hatch/skills/magic-moment/reference/design.md`。保证有来源的内容在手机尺度下清晰可读。每张卡片只有一个内容焦点，删除没有依据的字段。使用组件库自带的 Optimistic 字体。创作型卡片在 1240px 宽度下自包含，可见文字至少 32px，主行 56px。在 `/opt/hatch/skills/magic-moment/reference/design.md` 的单一表面规则下保留有意义的内部结构。对从放大中受益且有依据的展示使用 `tap: true`；普通卡片高度预算 600px，可点按卡片 1100px。

Author HTML and CSS only. Do not embed scripts, event handlers, iframes, or network dependencies. Reference local media with absolute `file://` paths. The authored capture mode disables page scripts and blocks network requests; real-app snapshot mode explicitly permits scripts. Do not combine authored HTML with `shell` or `hero`; its root supplies the surface.

只编写 HTML 和 CSS。不要嵌入脚本、事件处理器、iframe 或网络依赖。用绝对的 `file://` 路径引用本地媒体。创作捕获模式会禁用页面脚本并阻断网络请求；真实应用快照模式则明确允许脚本。不要把创作型 HTML 与 `shell` 或 `hero` 组合使用；它的根元素自身提供表面。

Include padding and borders within the card's declared width with
`box-sizing: border-box`. Let headings and status labels wrap or stack when
they cannot fit on one line. Check their full text in the rendered preview;
do not hide overflow to make an oversized layout pass.

用 `box-sizing: border-box` 把内边距和边框包含在卡片声明的宽度之内。当标题和状态标签一行放不下时，让它们换行或堆叠。在渲染预览中检查它们的完整文字；不要隐藏溢出内容让超尺寸的布局蒙混过关。

Time finite CSS actions to finish within the beat. Capture runs on one monotonic timeline through the beat endpoint, so finite actions finish once while ambient loops continue. Do not make a completed action loop. Inspect intermediate states for unintended overlap; a correct final state does not prove the transition works.

为有限的 CSS 动效计时，使其在节拍内完成。捕获沿一条单调时间线运行直至节拍终点，因此有限动作只完成一次，而环境循环继续进行。不要让已完成的动作进入循环。检查中间状态是否出现意外重叠；最终状态正确并不能证明过渡正常。

Keep the automatic avatar entrance, pinned header, and closing celebration. Use additional avatar media in cards only when the story is about generating that avatar. Preserve the creator's face in the final composition. The renderer uses a fixed overlay region, so verify face overlap visually rather than assuming automatic face detection.

保留自动的头像入场、固定页眉和结尾庆祝画面。只有当故事正是关于生成那个头像时，才在卡片中使用额外的头像媒体。在最终构图中保留创作者的面部。渲染器使用固定的叠加区域，所以要靠人工查看验证面部遮挡，而不要假设有自动人脸检测。
