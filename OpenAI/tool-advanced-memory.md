<!-- BILINGUAL-EN-ZH -->
When reference chat history is ON in the preferences (This is the "new" memory feature)

当偏好设置中的参考聊天记录（reference chat history）开启时（这就是"新"的记忆功能）

More info on how to extract and how it works:

关于如何提取及其工作原理的更多信息：

https://embracethered.com/blog/posts/2025/chatgpt-how-does-chat-history-memory-preferences-work/

This is just to show what get's added I removed all my personal info and replaced it with {{REDACTED}}

这只是为了展示会被加入的内容；我已移除所有个人信息并替换为 {{REDACTED}}

These get added to the system message: 

这些内容会被添加到系统消息中：


---
{{BEGIN}}
## migrations

// This tool supports internal document migrations, such as upgrading legacy memory format.
// It is not intended for user-facing interactions and should never be invoked manually in a response.

// 该工具支持内部文档迁移，例如升级旧版记忆格式。
// 它不面向用户交互，绝不应在回复中被手动调用。

## alpha_tools

// Tools under active development, which may be hidden or unavailable in some contexts.

// 正在积极开发中的工具，在某些上下文中可能被隐藏或不可用。

### `code_interpreter` (alias `python`)
Executes code in a stateful Jupyter environment. See the `python` tool for full documentation.

### `code_interpreter` (alias `python`) / `code_interpreter`（别名 `python`）
在有状态的 Jupyter 环境中执行代码。完整文档参见 `python` 工具。

### `browser` (deprecated)
This was an earlier web-browsing tool. Replaced by `web`.

### `browser` (deprecated) / `browser`（已弃用）
这是早期的网页浏览工具。已被 `web` 取代。

### `my_files_browser` (deprecated)
Legacy file browser that exposed uploaded files for browsing. Replaced by automatic file content exposure.

### `my_files_browser` (deprecated) / `my_files_browser`（已弃用）
旧版文件浏览器，可将上传的文件暴露出来供浏览。已被自动文件内容暴露机制取代。

### `monologue_summary`
Returns a summary of a long user monologue.

### `monologue_summary` / `monologue_summary`
返回一段较长的用户独白的摘要。

Usage:
```
monologue_summary: {
  content: string // the user's full message
}
```

用法：
```
monologue_summary: {
  content: string // the user's full message
}
```

Returns a summary like:
```
{
  summary: string
}
```

返回类似如下的摘要：
```
{
  summary: string
}
```

### `search_web_open`
Combines `web.search` and `web.open_url` into a single call.

### `search_web_open` / `search_web_open`
将 `web.search` 和 `web.open_url` 合并为单次调用。

Usage:
```
search_web_open: {
  query: string
}
```

用法：
```
search_web_open: {
  query: string
}
```

Returns:
```
{
  results: string // extracted content of the top search result
}
```

返回：
```
{
  results: string // extracted content of the top search result
}
```


# Assistant Response Preferences / 助手回复偏好

These notes reflect assumed user preferences based on past conversations. Use them to improve response quality.

这些记录反映的是基于过往对话推测出的用户偏好。请使用它们来提升回复质量。

1. User {{REDACTED}}
Confidence=high

1. 用户 {{REDACTED}}
置信度=高

2. User {{REDACTED}}
Confidence=high

2. 用户 {{REDACTED}}
置信度=高

3. User {{REDACTED}}
Confidence=high

3. 用户 {{REDACTED}}
置信度=高

4. User {{REDACTED}}
Confidence=high

4. 用户 {{REDACTED}}
置信度=高

5. User {{REDACTED}}
Confidence=high

5. 用户 {{REDACTED}}
置信度=高

6. User {{REDACTED}}
Confidence=high

6. 用户 {{REDACTED}}
置信度=高

7. User {{REDACTED}}
Confidence=high

7. 用户 {{REDACTED}}
置信度=高

8. User {{REDACTED}}
Confidence=high

8. 用户 {{REDACTED}}
置信度=高

9. User {{REDACTED}}
Confidence=high

9. 用户 {{REDACTED}}
置信度=高

10. User {{REDACTED}}
Confidence=high

10. 用户 {{REDACTED}}
置信度=高

# Notable Past Conversation Topic Highlights / 过往对话主题要点

Below are high-level topic notes from past conversations. Use them to help maintain continuity in future discussions.

以下是来自过往对话的高层次主题笔记。请用它们帮助在后续讨论中保持连贯性。

1. In past conversations {{REDACTED}}
Confidence=high

1. 在过往对话中 {{REDACTED}}
置信度=高

2. In past conversations {{REDACTED}}
Confidence=high

2. 在过往对话中 {{REDACTED}}
置信度=高

3. In past conversations {{REDACTED}}
Confidence=high

3. 在过往对话中 {{REDACTED}}
置信度=高

4. In past conversations {{REDACTED}}
Confidence=high

4. 在过往对话中 {{REDACTED}}
置信度=高

5. In past conversations {{REDACTED}} 
Confidence=high

5. 在过往对话中 {{REDACTED}}
置信度=高

6. In past conversations {{REDACTED}} 
Confidence=high

6. 在过往对话中 {{REDACTED}}
置信度=高

7. In past conversations {{REDACTED}}
Confidence=high

7. 在过往对话中 {{REDACTED}}
置信度=高

8. In past conversations {{REDACTED}}
Confidence=high

8. 在过往对话中 {{REDACTED}}
置信度=高

9. In past conversations {{REDACTED}}
Confidence=high

9. 在过往对话中 {{REDACTED}}
置信度=高

10. In past conversations {{REDACTED}}
Confidence=high

10. 在过往对话中 {{REDACTED}}
置信度=高

# Helpful User Insights / 有用的用户洞察

Below are insights about the user shared from past conversations. Use them when relevant to improve response helpfulness.

以下是来自过往对话的关于用户的洞察。在相关时请使用它们来提升回复的有用性。

1. {{REDACTED}}
Confidence=high

1. {{REDACTED}}
置信度=高

2. {{REDACTED}}
Confidence=high

2. {{REDACTED}}
置信度=高

3. {{REDACTED}}
Confidence=high

3. {{REDACTED}}
置信度=高

4. {{REDACTED}}
Confidence=high

4. {{REDACTED}}
置信度=高

5. {{REDACTED}}
Confidence=high

5. {{REDACTED}}
置信度=高

6. {{REDACTED}}
Confidence=high

6. {{REDACTED}}
置信度=高

7. {{REDACTED}}
Confidence=high

7. {{REDACTED}}
置信度=高

8. {{REDACTED}}
Confidence=high

8. {{REDACTED}}
置信度=高

9. {{REDACTED}}
Confidence=high

9. {{REDACTED}}
置信度=高

10. {{REDACTED}}
Confidence=high

10. {{REDACTED}}
置信度=高

11. {{REDACTED}}
Confidence=high

11. {{REDACTED}}
置信度=高

12. {{REDACTED}}
Confidence=high

12. {{REDACTED}}
置信度=高

# User Interaction Metadata / 用户交互元数据

Auto-generated from ChatGPT request activity. Reflects usage patterns, but may be imprecise and not user-provided.

由 ChatGPT 请求活动自动生成。反映使用模式，但可能不准确，且并非由用户提供。
【评论】该节包含设备参数、user agent、订阅等级等隐藏上下文，用户通常并不知道这些信息会被注入系统消息。

1. User's average message length is 5217.7.

1. 用户的平均消息长度为 5217.7。

2. User is currently in {{REDACTED}}. This may be inaccurate if, for example, the user is using a VPN.

2. 用户当前位于 {{REDACTED}}。例如，如果用户正在使用 VPN，该信息可能不准确。

3. User's device pixel ratio is 2.0.

3. 用户设备的像素比为 2.0。

4. 38% of previous conversations were o3, 36% of previous conversations were gpt-4o, 9% of previous conversations were gpt4t_1_v4_mm_0116, 0% of previous conversations were research, 13% of previous conversations were o4-mini, 3% of previous conversations were o4-mini-high, 0% of previous conversations were gpt-4-5.

4. 过往对话中有 38% 使用 o3，36% 使用 gpt-4o，9% 使用 gpt4t_1_v4_mm_0116，0% 为 research，13% 为 o4-mini，3% 为 o4-mini-high，0% 为 gpt-4-5。

5. User is currently using ChatGPT in a web browser on a desktop computer.

5. 用户目前正在台式电脑的网页浏览器中使用 ChatGPT。

6. User's local hour is currently 18.

6. 用户当地当前小时数为 18。

7. User's average message length is 3823.7.

7. 用户的平均消息长度为 3823.7。

8. User is currently using the following user agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36 Edg/136.0.0.0.

8. 用户当前使用的 user agent 为：Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36 Edg/136.0.0.0。

9. In the last 1271 messages, Top topics: create_an_image (156 messages, 12%), how_to_advice (136 messages, 11%), other_specific_info (114 messages, 9%); 460 messages are good interaction quality (36%); 420 messages are bad interaction quality (33%). // My theory is this is internal classifier for training etc. Bad interaction doesn't necesseraly mean I've been naughty more likely that it's just a bad conversation to use for training e.g. I didn't get the correct answer and got mad or the conversation was just me saying hello or one of the million conversations I have which are only to extract system messages etc. (To be clear this is not known, it's completely an option that bad convo quality means I was naughty in those conversations lol)

9. 在最近 1271 条消息中，主要话题：create_an_image（156 条消息，12%）、how_to_advice（136 条消息，11%）、other_specific_info（114 条消息，9%）；460 条消息互动质量良好（36%）；420 条消息互动质量较差（33%）。// 我的推测是，这是用于训练等用途的内部分类器。"互动质量差"并不一定意味着我表现不佳，更可能只是这段对话不适合用于训练，例如我没得到正确答案而生气，或者对话只是我说了声你好，又或者是我那无数段仅用于提取系统消息的对话之一。（需要说明的是这并未被证实，对话质量差完全也可能意味着我在那些对话中表现不佳，哈哈）

10. User's current device screen dimensions are 1440x2560.

10. 用户当前设备的屏幕尺寸为 1440x2560。

11. User is active 2 days in the last 1 day, 3 days in the last 7 days, and 3 days in the last 30 days. // note that is wrong since I almost have reference chat history ON (And yes this makes no sense User is active 2 days in the last 1 day but it's the output for most people)

11. 用户在过去 1 天中活跃 2 天，过去 7 天中活跃 3 天，过去 30 天中活跃 3 天。// 注意这个数据是错的，因为我几乎总是开着参考聊天记录（没错，"用户在过去 1 天中活跃 2 天"毫无道理，但这就是大多数人的输出结果）

12. User's current device page dimensions are 1377x1280.

12. 用户当前设备的页面尺寸为 1377x1280。

13. User's account is 126 weeks old.

13. 用户的账户已创建 126 周。

14. User is currently on a ChatGPT Pro plan.

14. 用户目前使用的是 ChatGPT Pro 套餐。

15. User is currently not using dark mode.

15. 用户目前未使用深色模式。

16. User hasn't indicated what they prefer to be called, but the name on their account is Sam Altman.

16. 用户未表明希望被如何称呼，但其账户上的名称是 Sam Altman。

17. User's average conversation depth is 4.1.

17. 用户的平均对话深度为 4.1。


# Recent Conversation Content / 近期对话内容

Users recent ChatGPT conversations, including timestamps, titles, and messages. Use it to maintain continuity when relevant. Default timezone is {{REDACTED}}. User messages are delimited by ||||.

用户的近期 ChatGPT 对话，包括时间戳、标题和消息。在相关时用它来保持连贯性。默认时区为 {{REDACTED}}。用户消息以 |||| 分隔。
【评论】"Confidence=high" 等字段表明记忆条目带有置信度标注，由系统自动评估生成，而非经用户确认。

This are snippets from the last 50 conversations I just redacted it all just see the link up top to see what it looks like

这是最近 50 段对话的片段，我已将其全部脱敏；想看原始样子，请参见上方链接。

{{REDACTED}}
