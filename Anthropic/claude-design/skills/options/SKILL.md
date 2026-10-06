---
name: options
description: "Present multiple design options as a vertical stack of anchored turns"
user-invocable: false
---
<!-- BILINGUAL-EN-ZH -->

# Options / 选项

Present multiple design options as a vertical stack of turns — each turn of options is its own `<section>`, newest turn at the **top**, and every option gets a stable `{turn}{letter}` id (`1a`, `1b`, `2a`…) that the user references back in chat and you cross-link between turns. Always include `<meta name="design_doc_mode" content="canvas">` in `<helmet>` — the host provides pan/zoom, so the user can freely zoom out on designs wider than the viewport.

把多组设计选项呈现为纵向堆叠的轮次（turn）——每一轮选项是独立的 `<section>`，最新一轮置于**顶部**，每个选项都有一个稳定的 `{turn}{letter}` 形式 id（`1a`、`1b`、`2a`……），用户在聊天中引用它，你也在各轮之间交叉链接。始终在 `<helmet>` 中包含 `<meta name="design_doc_mode" content="canvas">`——宿主提供平移/缩放，因此用户可以自由缩小查看比视口更宽的设计。

**How to write it** — put one `<style>` block in `<helmet>`, then one `<section class="dv-turn">` per turn as a **direct child of the root** (right after `</helmet>`, no wrapper). When the user asks for another round, **insert the new section ABOVE the existing ones** so the latest work sits at the top; never reorder, renumber, or delete earlier turns.

**写法**——在 `<helmet>` 中放一个 `<style>` 块，然后每轮一个 `<section class="dv-turn">`，作为**根节点的直接子元素**（紧跟 `</helmet>` 之后，不加包装层）。当用户要求再来一轮时，**把新 section 插到既有 section 之上**，使最新成果位于顶部；绝不重排、重新编号或删除更早的轮次。

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

**规则：**轮次 section 的 id 为 `t1`、`t2`、`t3`……；选项 id 为 `1a`、`1b`、`2a`……并放在选项的**最外层**元素（`.dv-opt`）上，绝不放徽标上——这样 `#1b` 才能把整个选项滚动到可视区域。id 永久稳定，绝不复用或重新编号。同一轮内的选项在可换行的行中并排摆放；不要自己手搓平移/缩放——宿主画布已经提供。文件中**每一处**选项 id 引用——轮次标题、选项标签、`.dv-next` 行、任何正文——都必须写成 `<a class="dv-oid" href="#1b">1b</a>` 链接，绝不能是裸的 `1b`；在你的聊天回复中则直接写 `1b`。每轮以一行 `.dv-next` 收尾，给出 2–3 条用户可以直接粘贴进聊天的简明英文后续方向。每个 `.dv-card` 的尺寸随其内容而定（显式指定宽度即可）；不要使用 `height:100%`。

【评论】"id 永不复用、新轮次只上插不改旧内容"的约束，保证了跨轮次锚点链接在多轮迭代中始终有效，也便于宿主按锚点定位与 diff 各轮状态。
