---
name: debug
description: Enable debug logging for this session and help diagnose issues
disable-model-invocation: true
---
<!-- BILINGUAL-EN-ZH -->

# Debug Skill / 调试技能

Help the user debug an issue they're encountering in this current Claude Code session.

帮助用户调试他们在当前这个 Claude Code 会话中遇到的问题。

## Debug Logging Just Enabled / 刚刚启用调试日志

Debug logging was OFF for this session until now. Nothing prior to this /debug invocation was captured.

直到此刻为止，本会话的调试日志始终处于关闭状态。此次 /debug 调用之前的内容一概未被捕获。

Tell the user that debug logging is now active at `~/.claude/debug/{{SESSION_ID}}.txt`, ask them to reproduce the issue, then re-read the log. If they can't reproduce, they can also restart with `claude --debug` to capture logs from startup.

告诉用户调试日志现已在 `~/.claude/debug/{{SESSION_ID}}.txt` 处生效，请其复现该问题，然后重新读取日志。如果无法复现，用户也可以改用 `claude --debug` 重启，以捕获从启动开始的日志。

## Session Debug Log / 会话调试日志

The debug log for the current session is at: `~/.claude/debug/{{SESSION_ID}}.txt`

当前会话的调试日志位于：`~/.claude/debug/{{SESSION_ID}}.txt`

No log file exists yet.

目前尚不存在日志文件。

For additional context, grep for [ERROR] and [WARN] lines across the full file.

需要更多上下文时，在整个文件中 grep [ERROR] 与 [WARN] 行。

## Daemon / 守护进程

No daemon lock or status file found — the background daemon does not appear to be running. If the issue involves background sessions or `claude agents`, the daemon log (if any) is at `~/.claude/daemon.log`.

未找到守护进程的锁文件或状态文件——后台守护进程似乎没有在运行。如果问题涉及后台会话或 `claude agents`，守护进程日志（如存在）位于 `~/.claude/daemon.log`。

## Issue Description / 问题描述

The user did not describe a specific issue. Read the debug log and summarize any errors, warnings, or notable issues.

用户没有描述具体问题。请阅读调试日志，总结其中的错误、警告或值得注意的问题。

## Settings / 设置

Remember that settings are in:

请记住，设置位于：

* user - ~/.claude/settings.json
  user（用户级）- ~/.claude/settings.json
* project - /private/tmp/skillcap/.claude/settings.json
  project（项目级）- /private/tmp/skillcap/.claude/settings.json
* local - /private/tmp/skillcap/.claude/settings.local.json
  local（本地级）- /private/tmp/skillcap/.claude/settings.local.json

## Instructions / 操作说明

1. Review the user's issue description
   审阅用户的问题描述
2. The last 20 lines show the debug file format. Look for [ERROR] and [WARN] entries, stack traces, and failure patterns across the file
   最后 20 行展示了调试文件的格式。在整个文件中查找 [ERROR] 与 [WARN] 条目、堆栈跟踪和失败模式
3. Consider launching the claude-code-guide subagent to understand the relevant Claude Code features
   考虑启动 claude-code-guide 子智能体，以了解相关的 Claude Code 功能
4. Explain what you found in plain language
   用平实的语言解释你的发现
5. Suggest concrete fixes or next steps
   给出具体的修复建议或后续步骤

【评论】设置一节硬编码的 `/private/tmp/skillcap/...` 路径表明，这份快照采集自一个临时安装环境而非标准用户目录，是泄漏样本来源的常见痕迹。
