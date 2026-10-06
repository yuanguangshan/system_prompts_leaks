---
name: "voice_selector"
description: "Provides the static system voice catalog used by Jarvis. It does not define a user-facing workflow."
metadata: { "includeInPrompt": false, "voiceOnly": true }
---
<!-- BILINGUAL-EN-ZH -->

# Voice Catalog Data / 语音目录数据

This directory exists only to ship `voice_source.json`, the system voice catalog
consumed by Jarvis. It does not define a user-facing voice workflow.

该目录仅用于分发 `voice_source.json`，即 Jarvis 所消费的系统语音目录。它不定义面向用户的语音工作流。

Do not use this skill to handle voice requests, choose voices, or create widgets.  
Follow the active agent instructions and use `muse.voice_options` for voice
selection and design.

不要使用此技能来处理语音请求、挑选语音或创建小组件。请遵循当前激活的智能体指令，并使用 `muse.voice_options` 进行语音的选择与设计。

【评论】front matter 中的 `includeInPrompt: false` 表明本文件不会注入运行时提示词，属于纯数据型技能目录。
