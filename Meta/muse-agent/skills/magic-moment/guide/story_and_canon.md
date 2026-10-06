<!-- BILINGUAL-EN-ZH -->

# Preserve the source story / 忠于源故事

Read the active request and its explicit corrections before selecting visuals. Use the supplied footage as the source. Preserve the full narration unless the user explicitly requests a shorter edit. Do not import unrelated music, style preferences, capabilities, or sample gallery copy from earlier work.

在选择视觉素材之前，先阅读当前请求及其显式更正。以提供的素材（footage）为源。除非用户明确要求更短的剪辑，否则保留完整旁白。不要从早期作品中引入无关的音乐、风格偏好、能力或示例文案。

Transcribe with `/opt/hatch/skills/magic-moment/mm transcribe`. Read the transcript and compare it with the user's supplied words. Preserve the ASR record. When the user supplies a correction, put `transcript_correction: {"text": "the corrected transcript", "source": "the user message or document that supplied it"}` in the screenplay. Corrected words have no automatic word alignment; inspect the footage to establish beat times.

用 `/opt/hatch/skills/magic-moment/mm transcribe` 进行转写。阅读转写文本并与用户提供的文字比对。保留 ASR 记录。当用户提供更正时，把 `transcript_correction: {"text": "the corrected transcript", "source": "the user message or document that supplied it"}` 写入剧本（screenplay）。被更正的词没有自动的词级对齐；需检视素材来确定节拍时间。

Before layout, list the source claims in `~/workspace/.output/<name>/script_review.json`. Record each claim's source span, actor, action, tense, evidence, and intended depiction. Preserve the difference between an offer, a plan, ongoing work, and a completed result. For example, "has planned runs for the last couple months" does not mean "will plan runs for the next two months".

排版之前，先在 `~/workspace/.output/<name>/script_review.json` 中列出源内容中的各项论断。记录每条论断的来源片段、主体、动作、时态、证据和预期的呈现方式。保留“要约、计划、进行中的工作、已完成的结果”之间的区别。例如，“过去几个月都有规划跑步”并不意味着“未来两个月会规划跑步”。

Collect the actual messages and artifacts referenced by the story. Read the relevant parent conversation through `chat.read_messages` when available. Inspect existing files under `~/workspace/your_files/` and the source artifact's own directory. Record observed details in the screenplay's `facts` object with their sources. Do not invent names, statistics, routes, approvals, transactions, or results to fill a component. Do not treat an author-written fact sheet as independent evidence.

收集故事所引用的真实消息与产物。在可用时通过 `chat.read_messages` 读取相关的父会话。检查 `~/workspace/your_files/` 下的现有文件以及源产物自身的目录。把观察到的细节连同其来源记录在剧本的 `facts` 对象中。不要为了填充某个组件而虚构姓名、统计数据、路线、批准、交易或结果。不要把创作者自己撰写的事实清单当作独立证据。

Use synthetic UI only to illustrate a supported claim when the original material is unavailable. Describe its origin in the review. Do not present it as a captured historical screen. Leave an unsupported detail out and let the narration carry the claim. Do not operate an app, change a plan, send a message, or regenerate user data merely to manufacture a receipt.

仅当原始素材不可用时，才用合成 UI 来图解有依据的论断。在 review 中说明其来历。不要把它呈现为真实截取的历史屏幕。没有依据的细节就直接舍弃，让旁白来承载该论断。不要为了制造一个“凭证”而去操作应用、更改套餐、发送消息或重新生成用户数据。

【评论】“不要为制造凭证而操作应用”是一条防虚构兼防副作用条款：禁止代理为产出内容而改动用户的真实账户状态。

Mechanical validation checks structure and timing. Review semantics yourself: every depicted claim needs source support, and every important narrated claim needs a depiction or an explicit decision to leave it in the voiceover. Do not describe mechanical validation as proof of story fidelity.

机械校验只检查结构与时间。语义审查要自己完成：每条被呈现的论断都要有来源支撑，每条重要的旁白论断要么有对应呈现，要么有“保留在旁白中”的明确决定。不要把机械校验描述为故事忠实性的证明。
