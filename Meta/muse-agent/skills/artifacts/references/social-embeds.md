---
description: Embedding social posts and reels (Instagram) and other third-party rich content in a web page. Which platforms can be framed, the exact embed markup, and the link-out degrade for surfaces that block iframes.
builders: web
---
<!-- BILINGUAL-EN-ZH -->

# Social posts, reels, and rich embeds / 社交帖子、reel 与富媒体嵌入

Read this when a page should show a social post, reel, or other third-party
rich content (a video, a player, an interactive widget) rather than just link
to it.

当页面需要展示社交帖子、reel 或其他第三方富内容（视频、播放器、交互式小部件），而不只是链接到它时，请阅读本文。

## What can and cannot be framed / 哪些内容可以嵌入 frame，哪些不可以

- **Instagram posts, reels, and tv**: append `/embed/` to the permalink path,
  giving `https://www.instagram.com/{p|reel|tv}/{id}/embed/`. This is the one
  sanctioned URL derivation; it renders the post with playback and works for
  logged-out viewers.
  **Instagram 帖子、reel 和 tv**：在永久链接路径后追加 `/embed/`，得到 `https://www.instagram.com/{p|reel|tv}/{id}/embed/`。这是唯一被认可的 URL 推导方式；它以可播放的形式渲染帖子，且对未登录的观看者有效。
- **Other platforms**: frame only an official embed endpoint built for it
  (YouTube's `/embed/{id}` is the common case). Verify the framed document
  actually renders the content for a logged-out viewer before shipping it; a
  platform that blocks framing or login-walls the embed gets a link-out card
  instead, and never try to bypass with a scraped player or a guessed URL.
  **其他平台**：只嵌入为其构建的官方 embed 端点（YouTube 的 `/embed/{id}` 是最常见的情况）。上线前先验证被嵌入的文档在未登录状态下确实能渲染内容；禁止嵌帧或对 embed 设置登录墙的平台，应改用链接跳转卡片，绝不要试图用抓取来的播放器或猜测的 URL 绕过。
  【评论】将 URL 推导严格限定为官方端点、禁止抓取播放器，这类约束既是对平台服务条款的遵从，也避免了构造出易失效或绕过平台限制的链接。
- **The user's own saved or liked Instagram posts**: their permalinks come
  from the Instagram skill, not from browsing the user's account pages. Read
  `/opt/hatch/skills/instagram/SKILL.md`: `instagram-cli saved-posts` returns each
  post's `url`, the permalink the derivation above starts from.
  **用户自己保存或点赞过的 Instagram 帖子**：其永久链接来自 Instagram 技能，而不是通过浏览用户的账户页面获取。请阅读 `/opt/hatch/skills/instagram/SKILL.md`：`instagram-cli saved-posts` 会返回每个帖子的 `url`，即上述推导所基于的永久链接。

## The markup / 标记写法

A card with a visible permalink anchor as the base layer, and the embed iframe
on top:

以一个可见的永久链接锚点作为底层卡片，并在其上叠加 embed iframe：

```html
<div class="embed-card"><!-- fixed-height box; reels read best near 9/16 -->
  <iframe src="https://www.instagram.com/reel/{id}/embed/"
          sandbox="allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox"
          referrerpolicy="no-referrer" loading="lazy" scrolling="no"></iframe>
  <a href="{permalink}" target="_blank" rel="noopener">Watch on Instagram</a>
</div>
```

- Use exactly that `sandbox` token set: it is the standard third-party-embed
  posture (the frame cannot reach your page, and its own links can still open).
  必须使用与此完全一致的 `sandbox` 令牌集合：这是标准的第三方嵌入安全姿态（该 frame 无法触达你的页面，而它自身的链接仍然可以打开）。
- Give the card a fixed height and a neutral skeleton background so the layout
  does not shift while the frame loads, and let the iframe absolutely fill it.
  为卡片设置固定高度和中性的骨架背景，使 frame 加载期间布局不发生偏移，并让 iframe 以绝对定位填满卡片。
- `loading="lazy"` always; with many embeds on one page, mount each iframe only
  as it scrolls near the viewport — embed documents are heavy.
  始终使用 `loading="lazy"`；当一页上有多个嵌入时，只有当 iframe 滚动到视口附近时才挂载它——embed 文档的开销很大。
- Keep the permalink anchor visible under or beside the frame. It is the
  attribution, and on surfaces that block iframes it is the content.
  让永久链接锚点在 frame 下方或旁边保持可见。它承担署名作用，而在屏蔽 iframe 的展示面上它就是内容本身。

## Where it renders / 在哪里可以渲染

The published artifact page and its share link allow third-party iframes, so
the embed plays there. The in-chat preview and some host frames block all
iframes; there your card degrades to the anchor. Design for that: the embed is
an enhancement layered on a link-out card that is complete by itself, never the
only content in the slot.

已发布的工件页面及其分享链接允许第三方 iframe，因此嵌入内容可以在那里播放。聊天内预览和部分宿主 frame 会屏蔽所有 iframe；此时卡片退化为锚点链接。要为这种情况做设计：嵌入只是叠加在链接跳转卡片之上的增强层，卡片本身必须独立完整，嵌入永远不应是该位置的唯一内容。

## Media bytes / 媒体资源

- Never hotlink platform CDN media (`cdninstagram.com`, `fbcdn.net`, and kin)
  in an `<img>` or `<video>`: those URLs are signed and expiring, the audit
  fails them, and they 403 after you ship. If the card needs a visual before
  the frame loads, use a neutral skeleton, not a fetched thumbnail.
  绝不要在 `<img>` 或 `<video>` 中热链接平台 CDN 媒体（`cdninstagram.com`、`fbcdn.net` 及同类域名）：这些 URL 带签名且会过期，审计会将其判定为不合格，上线后它们会返回 403。如果卡片在 frame 加载前需要视觉占位，应使用中性骨架，而不是抓取缩略图。
- Do not extract or play a platform's MP4 directly; there is no durable URL
  for it. The embed iframe or the permalink is the playback path.
  不要直接提取或播放平台的 MP4；它没有持久可用的 URL。embed iframe 或永久链接才是播放路径。
