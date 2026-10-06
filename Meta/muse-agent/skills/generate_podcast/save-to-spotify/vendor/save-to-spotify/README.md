<!-- BILINGUAL-EN-ZH -->
# save-to-spotify CLI release pin / save-to-spotify CLI 版本锁定

This directory records the official `save-to-spotify` release used by the `save-to-spotify`
wrapper. Spotify publishes per-OS/arch zip assets; Jarvis (par-msl/hatch-image)
vendors the linux/amd64 asset, extracts the `save-to-spotify` executable, and installs it as
the trusted payload at `/opt/hatch-image/vendor/save-to-spotify-cli/save-to-spotify`.

本目录记录 `save-to-spotify` 包装器所使用的官方 `save-to-spotify` 发行版本。Spotify 按操作系统/架构发布 zip 资产；Jarvis（par-msl/hatch-image）将 linux/amd64 资产纳入供应商管理，解压出 `save-to-spotify` 可执行文件，并作为受信任载荷安装到 `/opt/hatch-image/vendor/save-to-spotify-cli/save-to-spotify`。

The Rust wrapper (`skills/crates/save-to-spotify-cli`) is the policy/privsep/auth boundary:
it reuses Spotify's shared authd-owned PKCE grant through an opaque Sentinel surrogate, gates
content commands with HITL, clears caller auth env, blocks `update`, restricts forwarded args
to a per-subcommand allowlist (`validate_args`), and delegates to the fixed executable path.
It does not fork or patch the upstream binary. See the crate's `DESIGN.md` for the full plan.

Rust 包装器（`skills/crates/save-to-spotify-cli`）是策略/权限隔离/认证边界：它通过一个不透明的 Sentinel 代理复用 Spotify 由 authd 持有的共享 PKCE 授权，用 HITL（人工审核）把关内容类命令，清空调用方的认证环境变量，阻止 `update`，把转发的参数限制在每个子命令各自的允许列表内（`validate_args`），并把执行委托给固定的可执行文件路径。它不 fork 也不给上游二进制打补丁。完整方案见该 crate 的 `DESIGN.md`。
【评论】把安全边界放在上游二进制之外而不修改上游，是供应商场景下常见的"最小信任"封装手法，便于随上游版本升级。

## SOURCE.toml fields / SOURCE.toml 字段
- `repo` / `rev` / `tag` / `version` — upstream repository and exact release pin.
  `repo` / `rev` / `tag` / `version` —— 上游仓库与精确的发行版锁定。
- `package` — package name the pin represents.
  `package` —— 该锁定所代表的包名。
- `artifact` — installed trusted payload name.
  `artifact` —— 安装后的受信任载荷名称。
- `release_url` — official release page.
  `release_url` —— 官方发布页。
- `release_asset` — official linux/amd64 asset (a `.zip`).
  `release_asset` —— 官方 linux/amd64 资产（一个 `.zip`）。
- `release_asset_sha256` — expected checksum of that asset (from the release's `.sha256`).
  `release_asset_sha256` —— 该资产的预期校验和（来自发行版的 `.sha256`）。
- `release_asset_archive_member` — the executable to extract from the zip.
  `release_asset_archive_member` —— 要从 zip 中解压出的可执行文件。

## Update flow / 更新流程
1. Pick the new official release.
   选定新的官方发行版。
2. Update `SOURCE.toml` (`tag`, `rev`, `version`, `release_url`, `release_asset`,
   `release_asset_sha256`).
   更新 `SOURCE.toml`（`tag`、`rev`、`version`、`release_url`、`release_asset`、`release_asset_sha256`）。
3. Update hatch-image's vendored executable + installer checksum to match.
   同步更新 hatch-image 内置的可执行文件与安装器校验和，使其保持一致。
4. Diff upstream `auth/` and `config/` for OAuth/constant changes (client_id, scope, auth and
   token URLs, redirect URI/port, backend URL); update the wrapper constants if they moved.
   对比上游 `auth/` 与 `config/` 的 OAuth/常量变更（client_id、scope、认证与令牌 URL、重定向 URI/端口、后端 URL）；如有变动则更新包装器常量。
5. Re-audit the CLI's flag surface (`--help`) against the wrapper's `validate_args()` allowlist:
   add any new/renamed flags the wrapper needs, and reject new read/write/exec or
   credential-substitution gadget flags.
   对照包装器的 `validate_args()` 允许列表重新审计 CLI 的旗标面（`--help`）：添加包装器所需的任何新增/改名旗标，并拒绝新增的读/写/执行或凭据替换类危险旗标。
6. From hatch-extensions: `cargo fmt --all` and `cargo test -p save-to-spotify-cli`.
   在 hatch-extensions 中运行 `cargo fmt --all` 和 `cargo test -p save-to-spotify-cli`。
7. Run a live smoke test (authorize → upload → `episodes status` READY).
   运行一次真实冒烟测试（授权 → 上传 → `episodes status` 为 READY）。
8. Run image payload/bundle/deploy gates; bump Jarvis `extensions.toml` to the new
   hatch-extensions rev.
   运行镜像载荷/打包/部署门禁；把 Jarvis 的 `extensions.toml` 提升到新的 hatch-extensions rev。

Rollback = revert the `SOURCE.toml` pin + hatch-image executable/checksum, then bump
`extensions.toml` back to the known-good rev.

回滚 = 恢复 `SOURCE.toml` 的锁定以及 hatch-image 的可执行文件/校验和，然后把 `extensions.toml` 调回已验证可用的 rev。

## Operational note / 运维提示
The backend can force-deprecate an out-of-date CLI via the `X-Min-CLI-Version` response
header. Monitor releases and keep the pin current so uploads don't start failing.

后端可通过 `X-Min-CLI-Version` 响应头强制弃用过旧的 CLI。要持续关注发行版并保持锁定版本最新，以免上传开始失败。
