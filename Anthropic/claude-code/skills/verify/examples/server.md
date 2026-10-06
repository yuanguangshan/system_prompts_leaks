<!-- BILINGUAL-EN-ZH -->
# Verifying a server/API change / 验证服务器/API 变更

The handle is `curl` (or equivalent). The evidence is the response.

操作手段是 `curl`（或等效工具），证据就是响应本身。

## Pattern / 模式

1. Start the server (background, with a readiness poll - see below)
  1. 启动服务器（后台运行，并做就绪轮询——见下文）
2. `curl` the route the diff touches, with inputs that hit the changed branch
  2. 用 `curl` 请求本次改动涉及的路由，输入需命中被修改的分支
3. Capture the full response (status + headers + body)
  3. 捕获完整响应（状态码 + 响应头 + 响应体）
4. Compare to expected
  4. 与预期进行比对

## Lifecycle / 生命周期

If there's a run-skill it handles this. If not:

如果有 run-skill，则由它负责处理。如果没有：

```bash
<start-command> &> /tmp/server.log &
SERVER_PID=$!
for i in {1..30}; do curl -sf localhost:PORT/health >/dev/null && break; sleep 1; done
# ... your curls ...
kill $SERVER_PID
```

No readiness endpoint? Poll the route you're about to test until it
stops returning connection-refused, then add a beat.

没有就绪检查端点？那就轮询你将要测试的路由，直到它不再返回连接被拒绝，再多等一小段时间。

## Worked example / 完整示例

**Diff:** adds a `Retry-After` header to 429 responses in `rateLimit.ts`.
**Claim (PR body):** "clients can now back off correctly."

**Diff：**在 `rateLimit.ts` 中为 429 响应添加 `Retry-After` 响应头。
**声明（PR 描述）：**"clients can now back off correctly."

**Inference:** hitting the rate limit should now return `Retry-After: <n>`
in the response headers. It didn't before.

**推断：**触发限流后，响应头中现在应返回 `Retry-After: <n>`。之前是没有的。

**Plan:**
1. Start server
2. Hit the rate-limited endpoint enough times to trigger 429
3. Check the 429 response has `Retry-After` header
4. Check the value is a positive integer

**计划：**
1. 启动服务器
  1. 启动服务器
2. Hit the rate-limited endpoint enough times to trigger 429
  2. 多次请求受限流保护的端点，触发 429
3. Check the 429 response has `Retry-After` header
  3. 检查 429 响应是否带有 `Retry-After` 响应头
4. Check the value is a positive integer
  4. 检查该值是否为正整数

**Execute:**
```bash
# trigger the limit - 10 fast requests, limit is 5/sec per the diff
for i in {1..10}; do curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/api/thing; done
# -> 200 200 200 200 200 429 429 429 429 429

# capture the 429 headers
curl -si localhost:3000/api/thing | head -20
# -> HTTP/1.1 429 Too Many Requests
# -> Retry-After: 12
# -> ...
```

**执行：**
```bash
# trigger the limit - 10 fast requests, limit is 5/sec per the diff
for i in {1..10}; do curl -s -o /dev/null -w "%{http_code}\n" localhost:3000/api/thing; done
# -> 200 200 200 200 200 429 429 429 429 429

# capture the 429 headers
curl -si localhost:3000/api/thing | head -20
# -> HTTP/1.1 429 Too Many Requests
# -> Retry-After: 12
# -> ...
```

**Verdict:** PASS - `Retry-After: 12` present, positive integer.

**结论：**通过——`Retry-After: 12` 存在，且为正整数。

## What FAIL looks like / 失败时的表现

- Header absent -> the diff didn't take effect, or you're not actually
  hitting the 429 path (check the status code first)
  - 响应头缺失 -> 改动未生效，或者你实际上并未命中 429 路径（先检查状态码）
- Header present but value is `NaN` / `undefined` / negative -> the
  logic is wrong
  - 响应头存在但值为 `NaN` / `undefined` / 负数 -> 逻辑有误
- You got 200s all the way through -> you never triggered the changed
  path. Tighten the request burst or check the rate limit config.
  - 全程都返回 200 -> 你从未触发被修改的路径。加大请求突发量，或检查限流配置。
