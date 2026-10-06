---
description: Text fixed at build time that a web page speaks aloud (read-aloud, narration, pronunciation, spoken prompts), recorded with the bundled TTS skill instead of the viewer's device voices.
builders: web
---
<!-- BILINGUAL-EN-ZH -->

# Spoken audio / 朗读音频

Record speech whose text is fixed at build time with the TTS skill: read
`/opt/hatch/skills/tts/SKILL.md` for commands, voice choice, and language.
For fixed text, use browser `speechSynthesis` only when the user asks for
device speech.

文本在构建时即已固定的语音，应使用 TTS 技能录制：命令、音色选择和语言请阅读 `/opt/hatch/skills/tts/SKILL.md`。对于固定文本，仅当用户明确要求使用设备语音时，才使用浏览器的 `speechSynthesis`。

Text entered, fetched, or generated after the build needs runtime speech:
use `speechSynthesis` for that actual text. Pre-recorded sample text does not
satisfy this use case.

构建之后才输入、获取或生成的文本需要运行时语音：对实际文本使用 `speechSynthesis`。预录制的示例文本不能满足这一用例。

For fixed text, make one recording for each unit the viewer can play on its
own, speaking exactly the text shown with it. Ship each recording as an owned
asset and play it from an `<audio>` element in the page markup, started by the
viewer, with only one playing at a time.

对于固定文本，为每个可独立播放的单元各录制一段音频，朗读内容须与该单元旁展示的文本完全一致。每段录音作为自有资源随页面发布，通过页面标记中的 `<audio>` 元素由观看者触发播放，且同一时间只播放一个。

If the TTS tool returns "Spoken audio can't be generated here", do not retry
or switch speech engines, and do not quote that response. Finish with
`web_artifacts.exit_build` using `status: "failure"` and explain in your own
words that the requested audio could not be produced.

如果 TTS 工具返回 "Spoken audio can't be generated here"，不要重试或切换语音引擎，也不要引用该响应。应以 `status: "failure"` 调用 `web_artifacts.exit_build` 结束构建，并用自己的话说明无法生成所请求的音频。

【评论】"不要引用该响应"是防止工具输出被原样转述给用户的设计：要求模型改述失败原因，避免固定话术被当作回答复制传播。

For other failures, apply the TTS skill's Handling Failures corrections that
can finish within the build, such as recopying a mismatched voice id or
fixing `--language` or `--format`.
Do not substitute a different valid voice or another speech engine, including
`speechSynthesis`. If synthesis still fails, finish with `web_artifacts.exit_build`
using `status: "failure"` and explain the error. Do not schedule delayed retries
or promise audio after the build finishes.

对于其他失败，应用 TTS 技能 "Handling Failures"（失败处理）中能在构建内完成的修正措施，例如重新复制不匹配的 voice id，或修正 `--language` 或 `--format`。不要改用另一个有效音色或别的语音引擎，包括 `speechSynthesis`。如果合成仍然失败，以 `status: "failure"` 调用 `web_artifacts.exit_build` 结束构建并说明错误。不要安排延迟重试，也不要承诺构建结束后再提供音频。
