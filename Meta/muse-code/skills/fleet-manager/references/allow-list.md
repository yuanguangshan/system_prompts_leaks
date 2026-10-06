<!-- BILINGUAL-EN-ZH -->

# Allow-list template / 白名单模板

Split the helper by subcommand, never allow the bare helper: read and steer
verbs may run without a prompt; anything that starts, interrupts, ends,
adopts, attaches, connects or forgets stays on the permission prompt. The
guards are soft, so this split is the safety. Replace `<fleet>` with the
absolute path the skill read gave you.

按子命令拆分该辅助脚本的授权，绝不允许裸命令本身：读取（read）和引导（steer）类动词可以不经提示直接运行；任何会启动、中断、结束、收编、附加、连接或遗忘的操作都保留在权限提示环节。这些防护是软性的，因此这种拆分本身就是安全边界。把 `<fleet>` 替换为技能读取结果给出的绝对路径。

## Allow without a prompt (read) / 免提示放行（读取）

```text
python3 <fleet> doctor *
python3 <fleet> detect *
python3 <fleet> context *
python3 <fleet> list *
python3 <fleet> machines
python3 <fleet> status *
python3 <fleet> read *
python3 <fleet> dialog *
python3 <fleet> resources *
python3 <fleet> fetch *
python3 <fleet> wait *
python3 <fleet> events *
```

## Allow without a prompt (steer) — optional, per taste / 免提示放行（引导）——可选，视个人偏好

```text
python3 <fleet> approve *
python3 <fleet> deny *
python3 <fleet> send * --keys *
```

No text-`send` glob: `send * …` cannot tell the notification form from
`--type` (flags may follow the text) and an allow match wins, so every
`send` that carries text stays on the prompt. `--automated` labels what is
typed; it grants nothing.

不放行携带文本的 `send` 通配：`send * …` 无法区分通知形式与 `--type`（标志可能出现在文本之后），而 allow 匹配优先生效，因此每条携带文本的 `send` 都会留在权限提示环节。`--automated` 只是给所输入的内容打标签，不授予任何权限。

## Always prompt (guarded) / 始终提示（受保护）

```text
python3 <fleet> open *
python3 <fleet> stop *
python3 <fleet> close *
python3 <fleet> adopt *
python3 <fleet> attach *
python3 <fleet> connect *
python3 <fleet> forget *
python3 <fleet> send * --type
```

## Claude Code settings template / Claude Code 设置模板

Claude Code matches `Bash(...)` rules by command prefix (`:*` means "and
anything after"), so each rule names the helper's path plus one verb.
Replace `<fleet>` with the absolute path and put the block in
`.claude/settings.json` (project) or `~/.claude/settings.json` (user):

Claude Code 按命令前缀匹配 `Bash(...)` 规则（`:*` 表示"以及其后的任何内容"），因此每条规则都写明辅助脚本的路径加一个动词。把 `<fleet>` 替换为绝对路径，并将该配置块放入 `.claude/settings.json`（项目级）或 `~/.claude/settings.json`（用户级）：

```json
{
  "permissions": {
    "allow": [
      "Bash(python3 <fleet> doctor:*)",
      "Bash(python3 <fleet> detect:*)",
      "Bash(python3 <fleet> context:*)",
      "Bash(python3 <fleet> list:*)",
      "Bash(python3 <fleet> machines)",
      "Bash(python3 <fleet> status:*)",
      "Bash(python3 <fleet> read:*)",
      "Bash(python3 <fleet> dialog:*)",
      "Bash(python3 <fleet> resources:*)",
      "Bash(python3 <fleet> fetch:*)",
      "Bash(python3 <fleet> wait:*)",
      "Bash(python3 <fleet> events:*)"
    ],
    "ask": [
      "Bash(python3 <fleet> open:*)",
      "Bash(python3 <fleet> send:*)",
      "Bash(python3 <fleet> stop:*)",
      "Bash(python3 <fleet> close:*)",
      "Bash(python3 <fleet> adopt:*)",
      "Bash(python3 <fleet> attach:*)",
      "Bash(python3 <fleet> connect:*)",
      "Bash(python3 <fleet> forget:*)"
    ]
  }
}
```

Muse has no settings-level `Bash(...)` allow-list: its settings file's
`permissions` member is a permission profile, not a list of command rules,
and a settings file that holds only the block above does not load. In Muse
the split above is applied at the permission prompt.

Muse 没有设置文件层面的 `Bash(...)` 白名单：其设置文件中的 `permissions` 成员是权限配置档案，而不是命令规则列表，仅包含上述配置块的设置文件不会被加载。在 Muse 中，上述拆分是在权限提示环节执行的。

【评论】该模板体现了最小权限思路：只把无副作用的读取动词列入 allow，而把所有改变外部状态的动词（open/stop/adopt/forget 等）留在 ask 列表，并以"防护是软性的"自警，说明作者意识到白名单机制本身可被绕过。
