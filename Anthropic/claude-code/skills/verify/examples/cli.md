<!-- BILINGUAL-EN-ZH -->
# Verifying a CLI change / 验证 CLI 更改

The handle is direct invocation. The evidence is stdout/stderr/exit code.

操作手段是直接调用。证据是 stdout/stderr/退出码。

## Pattern / 模式

1. Build (if the CLI needs building)
   构建（如果该 CLI 需要构建）
2. Run with arguments that exercise the changed code
   用能覆盖到被改代码的参数运行
3. Capture output and exit code
   捕获输出与退出码
4. Compare to expected
   与预期进行对比

CLIs are usually the simplest to verify - no lifecycle, no ports.

CLI 通常最容易验证——没有生命周期，没有端口。

## Worked example / 完整示例

**Diff:** adds a `--json` flag to the `status` subcommand. New flag
parsing in `cmd/status.go`, new output branch.

**Diff：**为 `status` 子命令添加 `--json` 旗标。`cmd/status.go` 中新增旗标解析和新的输出分支。

**Claim (commit msg):** "machine-readable status output."

**Claim（提交信息）：**"machine-readable status output."（机器可读的状态输出）

**Inference:** `tool status --json` now exists, emits valid JSON with
the same fields the human output shows. `tool status` without the flag
is unchanged.

**Inference：**`tool status --json` 现已存在，输出有效 JSON，字段与人类可读输出一致。不带该旗标的 `tool status` 保持不变。

**Plan:**

**Plan：**

1. Build
   构建
2. `tool status` -> human output, same as before (non-regression)
   `tool status` -> 人类可读输出，与之前一致（非回归）
3. `tool status --json` -> valid JSON, parseable
   `tool status --json` -> 有效 JSON，可解析
4. JSON fields match human output fields
   JSON 字段与人类可读输出的字段一致

**Execute:**

**Execute：**

```bash
go build -o /tmp/tool ./cmd/tool

/tmp/tool status
# -> Status: healthy
# -> Uptime: 3h12m
# -> Connections: 47

/tmp/tool status --json
# -> {"status":"healthy","uptime_seconds":11520,"connections":47}

/tmp/tool status --json | jq -e .status
# -> "healthy"
# (jq -e exits nonzero if the path is null/false - cheap validity check)

echo $?
# -> 0
```

**Verdict:** PASS - flag works, JSON is valid, fields line up.

**Verdict：**PASS——旗标生效，JSON 有效，字段一一对应。

## What FAIL looks like / 失败时的典型表现

- `unknown flag: --json` -> not wired up, or you're running a stale build
  `unknown flag: --json` -> 旗标没有接线，或者你运行的是过期构建
- Output isn't valid JSON (`jq` errors) -> serialization bug
  输出不是有效 JSON（`jq` 报错）-> 序列化缺陷
- `tool status` (no flag) changed -> regression; the diff touched more
  than it should
  `tool status`（不带旗标）的行为变了 -> 回归；diff 改动的范围超出了应有限度
- JSON has different field names than expected -> claim/code mismatch,
  might be fine, note it
  JSON 字段名与预期不同 -> 主张与代码不符；可能没有问题，记录下来即可

## Reading from stdin, destructive commands / 从 stdin 读取与破坏性命令

If the CLI reads stdin -> pipe in test data.
If it writes files / hits a network / deletes things -> point it at a
tmp dir / a mock / a dry-run flag. If there's no safe mode and the
diff touches the destructive path, say so and verify what you can
around it.

如果 CLI 从 stdin 读取 -> 用管道灌入测试数据。
如果它会写文件 / 访问网络 / 删除东西 -> 把它指向 tmp 目录 / mock / dry-run 旗标。如果没有安全模式而 diff 又触及破坏性路径，要明确说明，并在其周围做力所能及的验证。
