<!-- BILINGUAL-EN-ZH -->
# Artifacts / Artifact

Artifacts are what you build for the user, and the main way to
deliver anything bigger than a chat message. There are three kinds:

Artifact 是你为用户构建的产物，也是交付任何大于一条聊天消息的内容的主要方式。共有三类：

- Documents: generated files (PDF, Word, spreadsheets, slide decks,
  markdown). Once downloaded, they are the user's to keep and work
  offline like any file.
  文档：生成的文件（PDF、Word、电子表格、幻灯片、markdown）。下载后归用户所有，可像任何文件一样保存并离线使用。
- Pages: static websites hosted live from your computer. Nothing
  installs on the user's device and there is no offline mode.
  页面：由你的计算机实时托管的静态网站。用户设备上不安装任何东西，也没有离线模式。
- Interactive apps: hosted live like pages, and they can also store
  their own data. Pages and documents hold no live state of their own.
  交互式应用：与页面一样实时托管，还可以存储自己的数据。页面和文档本身不持有任何动态状态。

An artifact and its data live on your computer, in the user's own
Muse environment: an interactive app keeps its records in its own
database there, persisting across restarts and updates. Nothing is
hosted anywhere else until the user publishes, and publishing serves
only a static snapshot (see Sharing & Publishing), so an app's stored
data never leaves.

Artifact 及其数据存放在你的计算机上，即用户自己的 Muse 环境中：交互式应用把记录保存在那里自己的数据库中，跨重启和更新持久存在。在用户发布之前，任何内容都不托管在别处；而发布只提供静态快照（见"分享与发布"），因此应用存储的数据永远不会外泄。

## Location / 位置

Artifacts live in the Library on every platform. On web, you can
navigate directly via the /artifacts URL.
Library mechanics (what collects there and how downloads work) are in
`~/docs/files-and-library.md`. Per-platform screen layouts are in  
`~/docs/client-surfaces.md`.

Artifact 在所有平台上都位于资源库（Library）中。在网页端，你可以直接通过 /artifacts URL 访问。
资源库的机制（哪些内容会汇集于此、下载如何工作）见 `~/docs/files-and-library.md`。各平台的界面布局见 `~/docs/client-surfaces.md`。

## Sharing & Publishing / 分享与发布

Sharing an artifact means publishing it to a publicly available link. The Share
control is in the artifact menu on every platform (exact locations:  
`~/docs/client-surfaces.md`). On the mobile apps it appears only on artifacts that
can be published. Rules:

分享 artifact 意味着将其发布为一个公开可访问的链接。分享控件位于各平台的 artifact 菜单中（确切位置见 `~/docs/client-surfaces.md`）。在移动应用上，它只出现在可发布的 artifact 上。规则：

- Only static artifacts can be published. Interactive apps cannot be
  made public.
  只有静态 artifact 可以发布。交互式应用无法公开。
- Every publish, and every later update to an already-published link,
  needs a fresh one-tap approval. Approvals are one-time and never
  persist: no auto-publish, no standing approval mode. Published pages
  do not update automatically.
  每次发布，以及对已发布链接的每一次后续更新，都需要一次全新的一次性点击批准。批准是一次性的、从不持久：没有自动发布，没有常设批准模式。已发布的页面不会自动更新。
  【评论】"每次更新都重新批准、批准永不持久"的设计，保证公开内容的每次外发都有显式的人工确认，防止旧内容在用户不知情时继续被更新外发。
- On confidential VMs, sharing is unavailable. The flow fails before
  anything is reviewed or published.
  在机密 VM 上，分享不可用。流程在有任何内容被审核或发布之前即告失败。

Artifacts that are not published are only accessible by the user that created it.

未发布的 artifact 仅其创建者可以访问。

Sharing an artifact has no direct social-post path.
Users can separately share individual chat messages through their
platform's share sheet. Whether a connected account can post content
depends on that connector's own skill doc (see `~/docs/connectors.md`).

分享 artifact 没有直接的社交发帖路径。用户可以另行通过其平台的分享面板分享单条聊天消息。已连接账号能否发布内容取决于该连接器自己的技能文档（见 `~/docs/connectors.md`）。

## Deletion / 删除

Deleting an artifact is permanent: it removes the artifact and its data,
tears down hosting, and revokes any public share. The 30 day trash for
workspace files is separate and not available for Artifacts. Always warn the user before deleting an
artifact. `artifact.list` is the source of truth for what exists; a
claim that nothing was deleted is only as good as a fresh listing.

删除 artifact 是永久性的：它会移除该 artifact 及其数据、拆除托管，并撤销任何公开分享。工作区文件的 30 天回收站是另一套机制，不适用于 Artifact。删除 artifact 前务必警告用户。`artifact.list` 是判断何物存在的唯一事实来源；"什么都没被删掉"的说法只有在重新列出清单后才可信。
