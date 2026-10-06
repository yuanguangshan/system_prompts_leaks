<!-- BILINGUAL-EN-ZH -->
# Example: Web server / API / 示例：Web 服务器 / API

The distinguishing concern for servers is **lifecycle**: an agent needs to
start the server in the background, verify it's up, interact with it, then
cleanly shut it down. A foreground `npm start` that blocks the shell is
useless to an agent.

服务器类应用的特有问题是**生命周期**：agent 需要在后台启动服务器、确认它已就绪、与之交互，然后干净地关闭它。一个阻塞 shell 的前台 `npm start` 对 agent 毫无用处。

## Structure to follow / 应遵循的结构

A good server run skill has:

一份好的服务器运行技能应包含：

1. **Prerequisites & setup** - same as any project.
  1. **前提与安装**——与任何项目相同。
2. **Run** - the background-launch pattern (below), not a blocking command.
  2. **运行**——后台启动模式（见下文），而不是阻塞式命令。
3. **Verify** - a `curl` or similar that confirms the server is actually up.
  3. **验证**——用 `curl` 或类似手段确认服务器确实已启动。
4. **Stop** - how to cleanly terminate the background process.
  4. **停止**——如何干净地终止后台进程。

If the background-launch + readiness-poll + smoke-curl sequence is more
than a couple of lines, put it in a `smoke.sh` inside the skill directory
and have `SKILL.md` say "run the smoke script." One command, exit code
tells you if the server is healthy.

如果"后台启动 + 就绪轮询 + 冒烟 curl"的序列超过两三行，就把它放进技能目录里的 `smoke.sh`，并让 `SKILL.md` 写明"运行冒烟脚本"。一条命令，退出码即能告诉你服务器是否健康。

## Background-launch pattern / 后台启动模式

Don't write:

不要这样写：

> ```bash
> npm start
> ```

That blocks. Instead, show how to launch in the background, wait for
readiness, and find the PID later:

那样会阻塞。应改为展示如何后台启动、等待就绪、以及之后如何找到 PID：

> ```bash
> npm start &> /tmp/server.log &
> SERVER_PID=$!
>
> # Wait for the server to come up (adjust timeout/port as needed)
> for i in {1..30}; do
>   curl -sf http://localhost:3000/health > /dev/null && break
>   sleep 1
> done
> ```

Then the verification step:

然后是验证步骤：

> ```bash
> curl http://localhost:3000/health
> # -> {"status":"ok"}
> ```

And stopping:

以及停止：

> ```bash
> kill $SERVER_PID
> # $! is the npm wrapper's PID and npm doesn't forward SIGTERM to the
> # server it spawned - killing the port's listener is what reliably frees it:
> lsof -ti:3000 -sTCP:LISTEN | xargs -r kill
> ```

Prefer the captured PID or the port over `pkill -f "<pattern>"`. Broad
patterns like `pkill -f "next|vite|node"` match the agent's own command
line and can kill the session that ran them.

应优先使用捕获到的 PID 或端口，而不是 `pkill -f "<pattern>"`。`pkill -f "next|vite|node"` 这类宽泛模式会匹配到 agent 自己的命令行，可能杀掉执行它的那个会话。

## Details worth documenting / 值得记录的细节

- **Which port.** Make it explicit and say how to override it (`PORT=4000 npm start`).
  - **用哪个端口。**写明确，并说明如何覆盖（`PORT=4000 npm start`）。
- **What "ready" looks like.** A specific log line or a health endpoint to hit.
  - **"就绪"长什么样。**某条具体的日志行，或一个可访问的健康检查端点。
- **Required env vars.** Database URL, API keys, etc. - with a template `.env`
  if the list is long.
  - **必需的环境变量。**数据库 URL、API 密钥等——如果清单很长，附一个 `.env` 模板。
- **Hot reload vs production mode.** If they differ meaningfully, say which
  to use and when.
  - **热重载与生产模式。**如果二者有明显差异，说明何时用哪个。
- **Dependent services.** If the server needs Redis/Postgres/etc., either
  point at a docker-compose that brings them up, or include the `docker run`
  command directly.
  - **依赖服务。**如果服务器需要 Redis/Postgres 等，要么指向一个能拉起它们的 docker-compose，要么直接给出 `docker run` 命令。

## Example snippet / 示例代码片段

Here's what a Run section for a typical Node API might look like:

下面是一个典型 Node API 的 Run 小节可能的样子：

> ## Run
>
> ## 运行
>
> Start the dev server in the background:
>
> 在后台启动开发服务器：
>
> ```bash
> npm run dev &> /tmp/api.log &
> ```
>
> The server listens on port 3000. Wait for it to be ready, then verify:
>
> 服务器监听 3000 端口。等待它就绪，然后验证：
>
> ```bash
> for i in {1..20}; do
>   curl -sf http://localhost:3000/health && break
>   sleep 0.5
> done
> curl http://localhost:3000/health
> # -> {"status":"ok","version":"1.2.3"}
> ```
>
> Logs are at `/tmp/api.log`. Stop by killing the port's listener (`$!`
> after `npm run dev &` is the npm wrapper, and npm doesn't forward
> SIGTERM to the server it spawned):
>
> 日志位于 `/tmp/api.log`。通过杀掉占用端口的监听进程来停止（`npm run dev &` 之后的 `$!` 是 npm 包装进程，npm 不会把 SIGTERM 转发给它派生的服务器）：
>
> ```bash
> lsof -ti:3000 -sTCP:LISTEN | xargs -r kill
> ```
>
> ### Environment / 环境变量
>
> | Variable | Required | Default | Notes |
> |---|---|---|---|
> | `DATABASE_URL` | Yes | - | Postgres connection string |
> | `PORT` | No | `3000` | |
> | `LOG_LEVEL` | No | `info` | `debug` / `info` / `warn` / `error` |
>
> | 变量 | 必需 | 默认值 | 说明 |
> |---|---|---|---|
> | `DATABASE_URL` | 是 | - | Postgres 连接字符串 |
> | `PORT` | 否 | `3000` | |
> | `LOG_LEVEL` | 否 | `info` | `debug` / `info` / `warn` / `error` |
