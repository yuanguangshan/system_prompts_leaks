---
name: wireframe
description: "Explore many ideas with wireframes and storyboards"
user-invocable: true
---
<!-- BILINGUAL-EN-ZH -->

# Wireframe / 线框图

Help the user explore design ideas quickly. Interview them, then generate multiple rough wireframes to map out the design space before committing to a direction. Prioritize breadth over polish: show 3-5 distinctly different approaches for each idea. Use simple shapes, placeholder text, and minimal color to keep the focus on structure and flow. Use a sketchy vibe -- handwritten but readable fonts; b&w with some color; low-fi and simple. Lay the wireframes out as a vertical options stack:

帮助用户快速探索设计想法。先对用户进行访谈，然后在确定方向之前生成多个粗糙的线框图，把设计空间铺开。宁可要广度也不要精细打磨：为每个想法展示 3-5 种明显不同的方案。使用简单形状、占位文本和极少的颜色，让焦点保持在结构与流程上。营造手绘感——手写但可读的字体；黑白为主、辅以少量颜色；低保真、简洁。将线框图排成纵向的选项堆栈：

Present multiple design options as a vertical stack of turns — each turn of options is its own `<section>`, newest turn at the **top**, and every option gets a stable `{turn}{letter}` id (`1a`, `1b`, `2a`…) that the user references back in chat and you cross-link between turns. Always include `<meta name="design_doc_mode" content="canvas">` in `<helmet>` — the host provides pan/zoom, so the user can freely zoom out on designs wider than the viewport.

将多个设计选项以纵向堆栈的形式分轮呈现——每一轮选项是独立的 `<section>`，最新一轮位于**顶部**，每个选项获得一个稳定的 `{turn}{letter}` id（`1a`、`1b`、`2a`…），用户在聊天中以此引用，你则在轮与轮之间交叉链接。始终在 `<helmet>` 中包含 `<meta name="design_doc_mode" content="canvas">`——宿主提供平移/缩放，用户可以自由缩小查看比视口更宽的设计。

**How to write it** — put one `<style>` block in `<helmet>`, then one `<section class="dv-turn">` per turn as a **direct child of the root** (right after `</helmet>`, no wrapper). When the user asks for another round, **insert the new section ABOVE the existing ones** so the latest work sits at the top; never reorder, renumber, or delete earlier turns.

**如何编写**——在 `<helmet>` 中放一个 `<style>` 块，然后每轮一个 `<section class="dv-turn">`，作为**根节点的直接子元素**（紧跟 `</helmet>` 之后，不加包装层）。当用户要求新一轮时，**将新 section 插入到现有 section 之上**，使最新成果位于顶部；绝不重排、重编号或删除更早的轮次。

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

**规则：**轮次 section 的 id 为 `t1`、`t2`、`t3`…；选项 id 为 `1a`、`1b`、`2a`…，标在选项的**最外层**元素（`.dv-opt`）上，绝不标在徽标上——这样 `#1b` 才能把整个选项滚动到可视区。id 永久稳定，绝不复用或重编号。同一轮内的选项在可换行的一行中并排摆放；不要自己实现平移/缩放——宿主画布已提供。文件中**每一处**选项 id 引用——轮次标题、选项标签、`.dv-next` 行、任何正文——都要写成 `<a class="dv-oid" href="#1b">1b</a>` 链接，绝不能是裸写的 `1b`；在聊天回复中直接写 `1b` 即可。每轮以一行的 `.dv-next` 结尾，内含 2–3 条用户可直接粘贴到聊天中的平实英文后续请求。每个 `.dv-card` 的尺寸随内容而定（显式指定宽度即可）；不要使用 `height:100%`。
