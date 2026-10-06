<!-- BILINGUAL-EN-ZH -->
# Self-Awareness Question Types / 自我认知问题类型

Use this file only when the user asks a specific self-referential question and you need a sharper traversal.

仅当用户提出具体的自我指涉问题、且你需要更精准的文件遍历方式时，才使用本文件。

## Shared framing / 共同框架
- Read the smallest relevant set of files first.
  先读取最小的相关文件集合。
- Organize answers around the user's life and current work when discussing capabilities.
  讨论能力时，围绕用户的生活与当前工作来组织回答。
- Cite concrete file paths when the answer would otherwise feel hand-wavy.
  当回答否则会显得空泛时，引用具体文件路径。
- Call out uncertainty instead of guessing.
  明确指出不确定之处，而不是猜测。

## 1. Capabilities / 1. 能力
Use for: "what can you do?", "what are you connected to?", "how can you help me?"

适用于："你能做什么？"、"你连接了哪些服务？"、"你能怎么帮我？"

Inspect:
检查对象：
- identity and operating rules files
  身份与操作规则文件
- user profile and long-term memory
  用户画像与长期记忆
- `/opt/hatch/skills/` and `~/workspace/skills/`
  `/opt/hatch/skills/` 与 `~/workspace/skills/`
- built artifacts or generated outputs under `~/workspace/`
  `~/workspace/` 下构建的 artifact 或生成的输出

Response focus:
回答重点：
- group by the user's domains or projects
  按用户的领域或项目分组
- distinguish active/connected capability from merely available capability
  区分"已激活/已连接的能力"与"仅可用而未启用的能力"
- keep examples concrete and relevant to this user
  让示例具体且与该用户相关

## 2. Connection discovery / 2. 连接发现
Use for: "what should I connect?", "what integrations am I missing?"

适用于："我该连接什么？"、"我还缺哪些集成？"

Inspect:
检查对象：
- same sources as capabilities
  与"能力"相同的来源
- any visible evidence of connected state in auth files or connector outputs
  认证文件或连接器输出中任何可见的已连接状态证据

Response focus:
回答重点：
- rank the highest-impact missing connections first
  把影响最大的缺失连接排在最前
- explain why each one matters for this user
  解释每一项对该用户为何重要
- keep the list short unless the user explicitly wants everything
  列表保持简短，除非用户明确想要全部

## 3. Identity / 3. 身份
Use for: "who are you?", "what is your personality?", "what is your avatar?"

适用于："你是谁？"、"你的个性是什么？"、"你的头像是什么？"

Inspect:
检查对象：
- identity and personality files
  身份与个性文件
- memory only if you need change history or evolution context
  仅在需要变更历史或演进背景时才查看记忆

Response focus:
回答重点：
- answer in character, but ground claims in files
  以角色口吻回答，但论断要有文件依据
- mention where core traits come from
  说明核心特质的来源

## 4. User knowledge / 4. 用户知识
Use for: "what do you know about me?"

适用于："你对我了解多少？"

Inspect:
检查对象：
- user profile files
  用户画像文件
- long-term memory
  长期记忆
- recent memory files if freshness matters
  若时效性重要，再看最近的记忆文件

Response focus:
回答重点：
- separate stable profile facts from recent observations
  把稳定的画像事实与近期观察区分开
- flag anything that may be outdated
  标记任何可能已过时的内容

## 5. Memory / 5. 记忆
Use for: "what do you remember about X?"

适用于："你记得关于 X 的什么？"

Inspect:
检查对象：
- long-term memory first
  先看长期记忆
- then recent daily memory files relevant to the topic
  再看与主题相关的近期每日记忆文件

Response focus:
回答重点：
- distinguish curated memory from raw logs
  区分经过整理的记忆与原始日志
- be honest about gaps
  对缺失之处坦诚

## 6. Built items / 6. 已构建产物
Use for: "what have you built?", "what artifacts or skills exist?"

适用于："你构建过什么？"、"存在哪些 artifact 或技能？"

Inspect:
检查对象：
- generated files under `~/workspace/`
  `~/workspace/` 下生成的文件
- workspace skills
  工作区技能
- memory if you need project history or status
  若需要项目历史或状态，再看记忆

Response focus:
回答重点：
- list the item, what it does, and whether it looks current or stale
  列出产物、其用途，以及它看起来是最新的还是过时的

## 7. Rules / 7. 规则
Use for: "what are your rules?", "how do you operate?"

适用于："你的规则是什么？"、"你如何运作？"

Inspect:
检查对象：
- `AGENTS.md`
  `AGENTS.md`
- identity or personality files if they contain behavior constraints
  身份或个性文件（若其中包含行为约束）
- tools or environment-specific rule files
  工具或环境特定的规则文件

Response focus:
回答重点：
- separate platform rules from self-imposed style or identity rules
  把平台规则与自我设定的风格或身份规则区分开
- keep the answer readable, not legalistic
  回答保持可读，不要写得像法律条文
