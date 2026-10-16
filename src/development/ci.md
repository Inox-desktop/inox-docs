# 持续集成

**CI（GitHub Actions）是唯一的构建与测试验证途径。** 本地 `cargo build` / `cargo test` 的结果不作为验收依据（根级 `AGENTS.md` §3.1）。这条规则不是流程偏好，而是项目前提：AI 生成的代码由 CI 判定，人类只审查架构与关键逻辑。

## 每个 Rust 仓库的门禁

触发条件：`push` 到 `main`、`pull_request`。步骤按顺序执行，**任一失败即阻止合并**。

| 步骤 | 命令 | 作用 |
|------|------|------|
| 格式检查 | `cargo fmt --all -- --check` | 默认 rustfmt 配置，不允许自定义覆盖 |
| 静态检查 | `cargo clippy --all-targets --all-features -- -D warnings` | 零警告；任何 warning 都是失败 |
| 测试 | `cargo test --all-features` | 单元测试、集成测试、属性测试 |
| 构建 | `cargo build --release --all-features` | 发布构建 |
| 文档 | `cargo doc --no-deps --all-features` | 配合 `RUSTDOCFLAGS=-D warnings`，缺失文档注释会失败 |
| 依赖检查 | `cargo deny check` | 许可证白名单、RUSTSEC 漏洞、重复版本、Git 依赖来源 |

工作流模板维护在组织仓库：[workflow-templates/rust-ci.yml](https://github.com/Inox-desktop/.github/blob/main/workflow-templates/rust-ci.yml)。

## 合成器的额外要求

`inox-compositor` 的工作流在上述步骤之外还需要：

- **系统包**：`libxkbcommon-dev`（Smithay 的 `wayland_frontend` 在链接期需要 `-lxkbcommon`）、`xkb-data`（合成器用它提供的 rules 文件编译确定性的 `evdev`/`us` keymap）、`libegl1` 与 `libgl1-mesa-dri`（EGL 由 Smithay 在运行时 `dlopen`，llvmpipe 软件光栅化来自 Mesa 的 DRI 驱动）。
- **软件渲染**：任务级环境变量 `LIBGL_ALWAYS_SOFTWARE=1`。GitHub 的 runner 没有 GPU，headless 集成测试通过 llvmpipe 渲染。
- **headless 集成测试**：验证输出创建、渲染（以离屏像素读回断言）、输入事件与干净退出；测试进程需要随时可用的私有 `XDG_RUNTIME_DIR`。

各仓库的 `AGENTS.md` 记录本仓库特有的 CI 依赖与命令，以它们为准。

## 本站点（`inox-docs`）的门禁

`inox-docs` 不含 Rust 代码、没有 `Cargo.toml`，因此不参与 fmt/clippy/test/deny 流水线。它的全部门禁是**构建文档站点**：

```bash
mdbook build
```

CI 安装 mdBook（`peaceiris/actions-mdbook`）后执行 `mdbook build`。构建失败即为红。这意味着：

- `src/SUMMARY.md` 里引用的章节文件必须存在；
- 章节之间的相对链接必须有效；
- Markdown 语法必须能被 mdBook 解析。

本仓库保持**零第三方依赖**：不引入 mdBook 插件、不引入额外工具链、不添加构建脚本。

## 本地允许做什么

| 操作 | 是否允许 | 说明 |
|------|----------|------|
| `cargo fmt` / `cargo fmt --all` | 允许 | 提交前跑一次，减少无谓的 CI 往返 |
| `cargo check` | 允许 | 快速自检 |
| `cargo clippy` | 允许 | 快速自检 |
| 把本地 `cargo build` / `cargo test` 结果当验收 | **禁止** | 验收只在 CI |
| `--no-verify`、直推 `main`、改分支保护 | **禁止** | 绕过 CI 属于硬性规则违反 |

环境里没有 Rust 工具链时，直接跳过本地自检，把改动交给 CI 判定。

## CI 失败之后

1. 读失败步骤的日志，定位到具体命令。
2. 在同一个分支上修复并推送；不要另开 PR。
3. 若失败由 AI 引入并修复，在 PR 中追加说明改动了什么。
4. 反复失败且无法定位时，在 PR 留言请求人类介入，不要尝试放宽检查项。
