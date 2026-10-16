# 组件仓库

Inox 采用多仓库（polyrepo）策略：每个组件一个独立的 GitHub 仓库，独立版本、独立 CI，跨仓库依赖通过锁定 `tag` 的 Git 依赖管理（见[依赖管理](../development/dependencies.md)）。仓库清单以 PRD §3.1 为准。

## 状态说明

本页的「当前状态」按以下口径标注，均可通过对应仓库核对：

| 标注 | 含义 |
|------|------|
| **已实现** | 仓库中已有可用实现，且有对应测试与 CI |
| **骨架** | 只有仓库结构、CI 与文档，`src/` 中尚无业务逻辑 |
| **计划中** | 仓库尚未创建，仅在 PRD 中规划 |

## 仓库清单

| 仓库 | 层级 | 职责 | 发布物 | 当前状态 |
|------|------|------|--------|----------|
| `.github` | 组织 | 组织级工作流模板、Issue/PR 模板、CONTRIBUTING | — | 已建立 |
| `inox` | 元仓库 | 项目总览、PRD、ADR、兼容矩阵、文档索引 | 文档 | 维护中 |
| `inox-common` | L1 | 共享类型、错误处理、日志、工具函数 | crate | 已实现（`v0.1.0` 已发布） |
| `inox-config` | L1 | 配置模型、TOML 解析、默认配置 | crate | 骨架 |
| `inox-docs` | L1 | 用户与开发者文档站点 | mdBook 站点 | 已实现（本站点） |
| `inox-compositor` | L2 | 基于 Smithay 的 Wayland 合成器 | 二进制 `inox-compositor` | 已实现（M1、M2） |
| `inox-protocols` | L2 | 标准协议绑定与自定义 Wayland 协议扩展 | crate | 骨架 |
| `inox-xwayland` | L2 | XWayland 集成 | 二进制/服务 | 骨架 |
| `inox-shell` | L3 | Shell 主框架与插件系统 | 二进制 `inox-shell` | 骨架 |
| `inox-panel` | L3 | 面板（任务栏、系统托盘、时钟） | 二进制 `inox-panel` | 骨架 |
| `inox-launcher` | L3 | 应用启动器 | 二进制 `inox-launcher` | 骨架 |
| `inox-notifications` | L3 | 通知守护进程 | 二进制 `inox-notifications` | 骨架 |
| `inox-settings` | L3 | 系统设置应用 | 二进制 `inox-settings` | 骨架 |
| `inox-lock` | L3 | 锁屏 | 二进制 `inox-lock` | 骨架 |
| `inox-wallpaper` | L3 | 壁纸设置与渲染 | 二进制 `inox-wallpaper` | 骨架 |
| `inox-session` | L4 | 会话管理器，拉起合成器与 Shell | 二进制 `inox-session` | 骨架 |
| `inox-idle` | L4 | 空闲管理 | 二进制 `inox-idle` | 骨架 |
| `inox-greet` | L4 | 显示管理器（可选） | 二进制 `inox-greet` | 骨架 |
| `inox-integration` | 测试 | 端到端集成测试，聚合所有组件 | 测试报告 | 骨架 |

规划中、**尚未创建**的仓库：`inox-portal`（`xdg-desktop-portal` 后端）、`inox-screenshot`（截图/录屏）、`inox-osd`（屏幕提示）、`inox-cli`（`inoxctl` 命令行工具）。

## 需要注意的几处

### `inox-common` 是叶子节点

`inox-common` 不得依赖任何其他 Inox crate，它是 L1 的依赖终点。因此它只收纳**不依赖 Inox 其他部分**的共享能力。当前已发布的 `v0.1.0` 提供 `logging` 模块（统一的 `tracing` 初始化入口）；XDG 基础目录解析（`paths` 模块）已在特性分支实现，计划随 `v0.2.0` 发布，尚未打 tag（见 [ADR 0009](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0009-configuration-model-and-reload.md)）。

`inox-common` 已被跨仓库消费：`inox-compositor` 通过锁定 `tag = "v0.1.0"` 的 Git 依赖引入它，因此任何破坏性变更都必须发布新 tag，不能在已发布的 tag 上重写历史。

### `inox-compositor` 的二进制名

PRD §3.1 把发布物记作 `inox-comp`；根级 `AGENTS.md` §5 说明这只是简写，实际二进制与你运行的命令都是 `inox-compositor`。

当前 `inox-compositor` 实现的是 **M1 + M2**：headless 输出创建、键盘与指针事件处理、以 `xdg-shell` toplevel 为窗口的 master/stack 平铺。它**不启用** DRM/KMS、libinput、udev 等后端特性，也无 XWayland、剪贴板与拖放。详见[构建与运行](../usage/building.md)。

### `inox-xwayland` 的状态

`inox-xwayland` 在 `main` 上仍是 M0 骨架。以「监管外部进程」方式接入 rootless XWayland 的 M1 工作在其特性分支上进行，尚未合入 `main`，因此本页把它标为骨架。

### 合成器与 Shell 之间没有 Rust 依赖

L2 与 L3 之间的边界不是「暂时没有依赖」，而是 ADR 0002 明确规定的设计：两者只能通过 Wayland 协议、`inox-ipc` 与 D-Bus 通信。这意味着 Shell 组件可以在虚拟合成器（`weston --backend=headless`）中单独测试，Shell 崩溃也不会带走合成器。

## 相关页面

- 每层的依赖规则见[总体架构](./overview.md)；
- 各组件的 CI 门禁见[持续集成](../development/ci.md)；
- 组件的已发布版本登记在元仓库的 [compat.toml](https://github.com/Inox-desktop/inox/blob/main/compat.toml)。
