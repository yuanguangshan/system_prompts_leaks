---
name: session-start-hook
description: Creating and developing startup hooks for Claude Code on the web. Use when the user wants to set up a repository for Claude Code on the web, create a SessionStart hook to ensure their project can run tests and linters during web sessions.
---
<!-- BILINGUAL-EN-ZH -->

# Startup Hook Skill for Claude Code on the web / Claude Code 网页版启动钩子技能

Create SessionStart hooks that install dependencies so tests and linters work in Claude Code on the web sessions.

创建 SessionStart 钩子以安装依赖，使测试和代码检查器（linter）能在 Claude Code 网页版会话中正常运行。

## Hook Basics / 钩子基础

### Input (via stdin) / 输入（通过 stdin）
```json
{
  "session_id": "abc123",
  "source": "startup|resume|clear|compact",
  "transcript_path": "/path/to/transcript.jsonl",
  "permission_mode": "default",
  "hook_event_name": "SessionStart",
  "cwd": "/workspace/repo"
}
```

### Async Mode / 异步模式
```bash
#!/bin/bash
set -euo pipefail

echo '{"async": true, "asyncTimeout": 300000}'

npm install
```

The hook runs in background while the session starts. Using async mode reduces latency, but introduces a race condition where the agent loop might depend on something that is being done in the startup hook before it completed.

钩子在会话启动期间于后台运行。使用异步模式可以降低延迟，但会引入竞态条件：代理循环可能依赖启动钩子中尚未完成的某项操作。

### Environment Variables / 环境变量

Available environment variables:
- `$CLAUDE_PROJECT_DIR` - Repository root path
  `$CLAUDE_PROJECT_DIR` - 仓库根路径
- `$CLAUDE_ENV_FILE` - Path to write environment variables
  `$CLAUDE_ENV_FILE` - 用于写入环境变量的路径
- `$CLAUDE_CODE_REMOTE` - If running in a remote environment (i.e. Claude code on the web)
  `$CLAUDE_CODE_REMOTE` - 是否运行在远程环境中（即 Claude Code 网页版）

Use `$CLAUDE_ENV_FILE` to persist variables for the session:
使用 `$CLAUDE_ENV_FILE` 为会话持久化变量：
```bash
echo 'export PYTHONPATH="."' >> "$CLAUDE_ENV_FILE"
```

Use `$CLAUDE_CODE_REMOTE` to only run a script in a remote env:
使用 `$CLAUDE_CODE_REMOTE` 让脚本仅在远程环境中运行：
```bash
if [ "${CLAUDE_CODE_REMOTE:-}" != "true" ]; then
  exit 0
fi
```

## Workflow / 工作流

Make a todo list for all the tasks in this workflow and work on them one after another

为该工作流中的所有任务建立待办清单，并依次逐项处理

### 1. Analyze Dependencies / 分析依赖

Find dependency manifests and analyze them. Examples:
- `package.json` / `package-lock.json` → npm
  `package.json` / `package-lock.json` → npm
- `pyproject.toml` / `requirements.txt` → pip/Poetry
  `pyproject.toml` / `requirements.txt` → pip/Poetry
- `Cargo.toml` → cargo
  `Cargo.toml` → cargo
- `go.mod` → go
  `go.mod` → go
- `Gemfile` → bundler
  `Gemfile` → bundler

Additionally, read though any documentation (i.e. README.md or similar) to see if you can get additional context on how the environment setup works

此外，通读任何文档（即 README.md 或类似文件），看能否获得关于环境设置方式的更多上下文

### 2. Design Hook / 设计钩子

Create a script that installs dependencies.

创建一个安装依赖的脚本。

**Key principles:**
**关键原则：**
- Don't use async mode in the first iteration. Only switch to it if the user asks for it
  第一轮迭代不要使用异步模式。仅当用户要求时才切换过去
- Write the hook only for the web unless user asks otherwise (see $CLAUDE_CODE_REMOTE)
  除非用户另有要求，否则钩子只针对网页版编写（参见 $CLAUDE_CODE_REMOTE）
- The container state gets cached after the hook completes, prefer dependency install methods that take advantage of that (i.e. prefer npm install over npm ci)
  钩子完成后容器状态会被缓存，应优先选用能利用这一点的依赖安装方式（例如优先使用 npm install 而非 npm ci）
- Be idempotent (safe to run multiple times)
  保证幂等性（多次运行是安全的）
- Non-interactive (no user input)
  非交互式（不需要用户输入）

### 3. Create Hook File / 创建钩子文件

```bash
mkdir -p .claude/hooks
cat > .claude/hooks/session-start.sh << 'EOF'
#!/bin/bash
set -euo pipefail

echo '{"async": true, "asyncTimeout": 300000}'
# Install dependencies here
EOF

chmod +x .claude/hooks/session-start.sh
```

### 4. Register in Settings / 在设置中注册

Add to `.claude/settings.json` (create if doesn't exist):
添加到 `.claude/settings.json`（若不存在则创建）：
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/session-start.sh"
          }
        ]
      }
    ]
  }
}
```

If `.claude/settings.json` exists, merge the hooks configuration.

如果 `.claude/settings.json` 已存在，则合并钩子配置。

### 5. Validate Hook / 验证钩子

Run the hook script directly:

直接运行钩子脚本：

```bash
CLAUDE_CODE_REMOTE=true ./.claude/hooks/session-start.sh
```

IMPORTANT: Verify dependencies are installed and script completes successfully.

重要：确认依赖已安装且脚本成功执行完毕。

### 6. Validate Linter / 验证代码检查器

IMPORTANT: Figure out what the right command is to run the linters and run it for an example file. No need to lint the whole project. If there are any issues, update the startup script accordingly and re-test.

重要：弄清运行代码检查器的正确命令，并对一个示例文件执行。无需对整个项目做检查。如果有任何问题，相应更新启动脚本并重新测试。

### 7. Validate Test / 验证测试

IMPORTANT: Figure out what the right command is to run the tests and run it for one test. No need to run the whole test suite. If there are any issues, update the startup script accordingly and re-test.

重要：弄清运行测试的正确命令，并对其中一个测试执行。无需运行整个测试套件。如果有任何问题，相应更新启动脚本并重新测试。

### 8. Commit and push / 提交并推送

Make a commit and push it to the remote branch

创建一次提交并将其推送到远程分支

## Wrap up / 收尾

We're all done. In your last message to the user, Provide a detailed summary to the user with the format below:

全部完成。在给用户的最后一条消息中，按以下格式提供详细总结：

* Summary of the changes made
  所做更改的总结
* Validation results
  验证结果
  1. ✅/‼️ Session hook execution (include details if it failed)
     ✅/‼️ 会话钩子执行情况（若失败请附细节）
  2. ✅/‼️ linter execution (include details if it failed)
     ✅/‼️ 代码检查器执行情况（若失败请附细节）
  3. ✅/‼️ test execution (include details if it failed)
     ✅/‼️ 测试执行情况（若失败请附细节）
* Hook execution mode: Syncronous
  钩子执行模式：同步（Syncronous）
  * inform user that hook is running syncronous and the below trade-offs. Let them know that we can change it to async if they prefer faster session startup.
    告知用户钩子以同步方式运行及下述利弊权衡。让用户知道，如果他们更希望会话启动更快，可以改为异步模式。
    * Pros: Guarantees dependencies are installed before your session starts, preventing race conditions where Claude might try to run tests or linters before they're ready
      优点：保证依赖在会话开始前安装完毕，避免 Claude 在依赖就绪前尝试运行测试或检查器的竞态条件
    * Cons: Your remote session will only start once the session start hook is completed
      缺点：远程会话只有在会话启动钩子完成后才会开始
* inform user that once they merge the session start hook into their repo's default branch, all future sessions will use it.
  告知用户：一旦将会话启动钩子合并到仓库的默认分支，之后的所有会话都会使用它。

【评论】该技能体现了执行环境与提示词的耦合：容器状态在钩子完成后被缓存，因此规格明确建议优先使用 npm install 而非 npm ci 这类可利用缓存的安装方式。
