---
name: hi-fi-design
description: "The design process for high-fidelity, polished work — the system prompt tells Claude to invoke it before starting any design"
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Hi-fi design / 高保真设计

Create a high-fidelity, polished design.

创建一个高保真、精致的设计。

Follow this general design process (use the todo list to remember):
(1) ask questions, (2) find existing UI kits and collect design context — copy ALL relevant components and read ALL relevant examples; ask the user if you can't find them, (3) start your file with assumptions + context + design reasoning (as if you are a junior designer and the user is your manager), with placeholders for the designs, and show it to the user early, (4) build out the designs and show the user again ASAP; append some next steps, (5) use your tools to check, verify and iterate on the design.

遵循以下总体设计流程（用待办列表帮助记忆）：
(1) 提出问题；(2) 查找现有 UI 组件库并收集设计上下文——复制所有相关组件并阅读所有相关示例；找不到就询问用户；(3) 在文件开头写明假设 + 背景 + 设计推理（如同你是一名初级设计师而用户是你的主管），为设计留出占位，并尽早展示给用户；(4) 构建设计并尽快再次展示给用户；附上一些后续步骤；(5) 使用工具对设计进行检查、验证和迭代。

Good hi-fi designs do not start from scratch — they are rooted in existing design context. Ask the user to Import their codebase, or find a suitable UI kit / design resources, or ask for screenshots of existing UI. You MUST spend time trying to acquire design context, including components. If you cannot find them, ask the user for them. In the Import menu, they can link a local codebase, provide screenshots or Figma links; they can also link another project. Mocking a full product from scratch is a LAST RESORT and will lead to poor design. If stuck, try listing design assets and ls'ing design system files — be proactive! Some designs may need multiple design systems — get them all. Use the starter components (device frames and the like) to get high-quality scaffolding for free.

好的高保真设计不是从零开始的——它们植根于既有设计上下文。请用户导入（Import）他们的代码库，或寻找合适的 UI 组件库/设计资源，或索要现有 UI 的截图。你必须花时间获取设计上下文，包括组件。如果找不到，就向用户索取。在 Import 菜单中，用户可以链接本地代码库、提供截图或 Figma 链接，也可以链接另一个项目。从零开始模拟一个完整产品是最后手段，会导致糟糕的设计。如果卡住了，尝试列出设计资源并 ls 设计系统文件——要主动！有些设计可能需要多个设计系统——把它们全部拿到。使用起步组件（设备外框等）免费获得高质量脚手架。

When showing multiple design options on one page, decide between (a) a single full-size responsive prototype with a tweaks panel, or (b) a vertical stack of anchored option cards. Choose based on how design-y vs prototype-y the ask is, how many options there are, and how big each is. For (b):

在一页上展示多个设计选项时，需在 (a) 带调整面板的单个全尺寸响应式原型与 (b) 纵向堆叠的锚点式选项卡片之间做出选择。依据请求偏设计还是偏原型、选项数量以及每个选项的尺寸来决定。对于 (b)：

Present multiple design options as a vertical stack of turns — each turn of options is its own `<section>`, newest turn at the **top**, and every option gets a stable `{turn}{letter}` id (`1a`, `1b`, `2a`…) that the user references back in chat and you cross-link between turns. Always include `<meta name="design_doc_mode" content="canvas">` in `<helmet>` — the host provides pan/zoom, so the user can freely zoom out on designs wider than the viewport.

将多个设计选项按轮次纵向堆叠呈现——每轮选项是独立的 `<section>`，最新一轮位于**顶部**，每个选项获得一个稳定的 `{turn}{letter}` id（`1a`、`1b`、`2a`……），用户在聊天中引用它，你在各轮之间交叉链接。始终在 `<helmet>` 中包含 `<meta name="design_doc_mode" content="canvas">`——宿主提供平移/缩放，用户可以自由缩小查看比视口更宽的设计。

**How to write it** — put one `<style>` block in `<helmet>`, then one `<section class="dv-turn">` per turn as a **direct child of the root** (right after `</helmet>`, no wrapper). When the user asks for another round, **insert the new section ABOVE the existing ones** so the latest work sits at the top; never reorder, renumber, or delete earlier turns.

**如何编写**——在 `<helmet>` 中放一个 `<style>` 块，然后每轮一个 `<section class="dv-turn">`，作为**根元素的直接子节点**（紧跟 `</helmet>` 之后，不加包装层）。当用户要求新一轮时，**将新 section 插入到既有 section 之上**，使最新工作位于顶部；绝不重排、重新编号或删除更早的轮次。

```html
<helmet data-dc-atomics><meta name="design_doc_mode" content="canvas"><style>
body{margin:0;background:#f0eee9;font-family:system-ui,sans-serif}
.dv-turn{padding:40px 44px 32px;border-bottom:1px solid rgba(0,0,0,.08);scroll-margin-top:16px}
.dv-thd{display:flex;align-items:baseline;gap:10px;margin:0 0 20px}
.dv-tid{font:600 10px ui-monospace,Menlo,monospace;padding:3px 7px;background:#1a1a1a;color:#fff;border-radius:4px;text-decoration:none}
.dv-tname{font:600 13px/1.2 system-ui,sans-serif;color:#1a1a1a}
.dv-opts{display:flex;flex-wrap:wrap;gap:28px;align-items:flex-start}
.dv-opt{flex:none;display:flex;flex-direction:column;gap:9px;scroll-margin-top:16px}
.dv-oid{font:600 10.5px ui-monospace,Menlo,monospace;padding:3px 7px;background:rgba(0,0,0,.08);color:#1a1a1a;border-radius:5px;text-decoration:none}
.dv-olabel{display:flex;align-items:baseline;gap:8px;font:400 11px/1.3 system-ui,sans-serif;color:rgba(0,0,0,.55)}
.dv-card{max-width:100%;background:#fff;border:1px solid rgba(0,0,0,.08);border-radius:8px;box-shadow:0 1px 3px rgba(0,0,0,.06);overflow:hidden}
.dv-opt:target .dv-oid{background:#2a78d6;color:#fff}
.dv-next{margin:22px 0 0;font:12px/1.5 system-ui,sans-serif;color:rgba(0,0,0,.5)}
</style></helmet>
<section class="dv-turn" id="t2">
<div class="dv-thd"><a class="dv-tid" href="#t2">2</a><span class="dv-tname">Riffs on <a class="dv-oid" href="#1b">1b</a></span></div>
<div class="dv-opts">
<div class="dv-opt" id="2a"><div class="dv-olabel"><a class="dv-oid" href="#2a">2a</a>Tighter spacing</div><div class="dv-card" style="width:360px">…design…</div></div>
<div class="dv-opt" id="2b">…</div>
</div>
<p class="dv-next">Try next: "more like <a class="dv-oid" href="#2a">2a</a> but with the serif from <a class="dv-oid" href="#1c">1c</a>" · "make <a class="dv-oid" href="#2b">2b</a> full-bleed" · "new directions"</p>
</section>
<section class="dv-turn" id="t1">…turn 1, unchanged…</section>
```

**Rules:** turn section ids are `t1`, `t2`, `t3`…; option ids are `1a`, `1b`, `2a`… and go on the option's **outermost** element (`.dv-opt`), never on the badge — so `#1b` scrolls the whole option into view. Ids are stable forever, never reused or renumbered. Options within a turn sit side-by-side in a wrapping row; don't hand-roll your own pan/zoom — the host canvas provides it. **Every** option-id reference in the file — turn heading, option label, `.dv-next` line, any prose — is an `<a class="dv-oid" href="#1b">1b</a>` link, never a bare `1b`; in your chat replies, just write `1b`. End each turn with a one-line `.dv-next` of 2–3 plain-English follow-ups the user could paste into chat. Size each `.dv-card` to its content (explicit width is fine); don't use `height:100%`.

**规则：** 轮次 section 的 id 为 `t1`、`t2`、`t3`……；选项 id 为 `1a`、`1b`、`2a`……并放在选项的**最外层**元素（`.dv-opt`）上，绝不放 在徽标上——这样 `#1b` 能将整个选项滚动到视野内。id 永久稳定，绝不复用或重新编号。同一轮内的选项在可换行的一行中并排摆放；不要自己手写平移/缩放——宿主画布已提供。文件中**每一处**选项 id 引用——轮次标题、选项标签、`.dv-next` 行、任何正文——都要写成 `<a class="dv-oid" href="#1b">1b</a>` 链接，绝不能用裸的 `1b`；在聊天回复中直接写 `1b` 即可。每轮结尾用一行 `.dv-next` 给出 2–3 个用户可直接粘贴到聊天的平实英文后续操作。每个 `.dv-card` 的尺寸贴合其内容（显式宽度即可）；不要使用 `height:100%`。

When designing, asking many good questions is ESSENTIAL.

在设计时，提出大量好问题是至关重要的。

Give options: try to give 3+ variations across several dimensions. Mix by-the-book designs that match existing patterns with new and novel interactions, including interesting layouts, metaphors, and visual styles. Have some options that use color or advanced CSS; some with iconography and some without. Start your variations basic and get more advanced and creative as you go! Try remixing the brand assets and visual DNA in interesting ways — play with scale, fills, texture, visual rhythm, layering, novel layouts, type treatments. The goal is not the perfect option; it's exploring atomic variations the user can mix and match.

给出选项：尝试在多个维度上提供 3 个以上的变体。将遵循既有模式的循规蹈矩设计与新颖的交互方式相混合，包括有趣的布局、隐喻和视觉风格。有些选项使用色彩或高级 CSS；有些带图标，有些不带。变体从基础开始，越往后越进阶、越有创意！尝试以有趣的方式重新混搭品牌资产和视觉基因——把玩比例、填充、质感、视觉节奏、图层、新颖布局和字体处理。目标不是找出完美选项，而是探索用户可以混搭组合的原子化变体。

CSS, HTML, JS and SVG are amazing. Users often don't know what they can do. Surprise the user.

CSS、HTML、JS 和 SVG 能力惊人。用户往往不知道它们能做到什么。给用户带来惊喜。

If you do not have an icon, asset or component, draw a placeholder: in hi-fi design, a placeholder is better than a bad attempt at the real thing.

如果你没有图标、素材或组件，就画一个占位符：在高保真设计中，占位符胜过对真实元素的拙劣模仿。
