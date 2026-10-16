# 总体架构

Inox 采用四层架构，**依赖只能向下**（`L4 → L3 → L2 → L1`），禁止反向依赖、跨层横向依赖与循环依赖。该约束由 [ADR 0002](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0002-layering-and-dependency-direction.md) 固定，条款见 PRD §2.1。

```text
┌─────────────────────────────────────────────┐
│ Layer 4: 会话与系统集成                      │
│  inox-session, inox-greet, inox-idle         │
├─────────────────────────────────────────────┤
│ Layer 3: Shell 与用户界面                    │
│  inox-shell, inox-panel, inox-launcher,      │
│  inox-notifications, inox-settings,          │
│  inox-lock, inox-wallpaper                   │
├─────────────────────────────────────────────┤
│ Layer 2: 合成器与窗口管理                    │
│  inox-compositor, inox-xwayland,             │
│  inox-protocols                              │
├─────────────────────────────────────────────┤
│ Layer 1: 基础设施与共享库                    │
│  inox-common, inox-config, inox-docs         │
└─────────────────────────────────────────────┘
```

## 各层职责

| 层级 | 职责 | 成员 |
|------|------|------|
| **L1** 基础设施与共享库 | 通用类型、错误处理、日志、配置解析、文档 | `inox-common`、`inox-config`、`inox-docs` |
| **L2** 合成器与窗口管理 | DRM/KMS、输入、渲染、窗口管理、协议绑定、XWayland 集成 | `inox-compositor`、`inox-protocols`、`inox-xwayland` |
| **L3** Shell 与用户界面 | 基于 Iced 的桌面外壳与 UI 组件 | `inox-shell`、`inox-panel`、`inox-launcher`、`inox-notifications`、`inox-settings`、`inox-lock`、`inox-wallpaper` |
| **L4** 会话与系统集成 | 会话管理、空闲管理、显示管理器 | `inox-session`、`inox-idle`、`inox-greet` |
| 元 / 组织 | 文档与组织级模板，不参与依赖图 | `inox`、`.github` |
| 测试 | 端到端集成测试 | `inox-integration` |

## 依赖规则

1. 依赖方向只能向下：`L4 → L3 → L2 → L1`。允许跨层向下（例如 L3 直接依赖 L1），禁止反向与跨层横向依赖。
2. `inox-common` 是**叶子节点**，不得依赖任何其他 Inox crate，也不得依赖 `inox-config`。
3. `inox-compositor` 与 `inox-shell`（及其 L3 子组件）之间**不建立 Rust 依赖**，只通过 Wayland 协议、`inox-ipc` 与 D-Bus 通信，二者必须能各自独立启动与独立测试。
4. `inox-protocols` 是协议的唯一出口：新增协议必须先在 `inox-protocols` 中定义或引用，并说明安全与生命周期约束（根级 `AGENTS.md` §4）。
5. L3 对 L2 的依赖有明确例外：`inox-wallpaper` 可依赖 `inox-compositor` 的渲染能力；其余 L3 组件不得直接依赖 `inox-compositor`。
6. `inox-integration` 是测试聚合仓库，不受上述方向限制，但只允许在各组件的发布 tag 上做验证。
7. 循环依赖、反向依赖与横向依赖由 CI 检查拦截（`cargo tree` 审计与 `cargo deny check` 的 `[bans]` 段）。

## 为什么这样分层

多仓库拆分本身并不消除耦合，只是把耦合从 crate 之间搬到仓库之间。ADR 0002 因此把依赖方向作为单条判据固定下来：任何一个仓库「可以引入什么依赖」只需看分层即可判断，无需逐个协商。这样做的收益是每个仓库都能独立编译、独立测试，且 L1 变更的影响面上界明确；代价是跨组件调用需要经过协议与序列化，且 L1 的改动要传播到 L4 需要多轮 tag 升级。

依赖图之外的硬性技术约束（Smithay 抽象、calloop 事件循环、Iced 的 Elm 架构）见 [ADR 0003](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0003-technology-stack.md)。

## 运行时数据流

```text
 应用 (Wayland/X11 客户端)
        │  wayland socket
        ▼
 ┌───────────────────┐   inox-ipc (自定义 Wayland 协议)
 │  inox-compositor  │◄───────────────────────────┐
 └───────────────────┘                             │
        ▲  layer-shell / xdg-shell              ┌──┴────────────┐
        │                                       │  inox-shell   │
 ┌──────┴────────┐   D-Bus (notifications,      │  及 L3 组件   │
 │  inox-session │   settings, portals)         └───────────────┘
 └───────────────┘
```

- **Wayland 协议**：客户端通过 `wayland socket` 连接合成器；Shell 组件使用 `wlr-layer-shell`、`xdg-shell` 等标准协议，以及 Inox 自定义的 `inox-ipc`（定义于 `inox-protocols`）。
- **D-Bus**：通知、设置、`xdg-desktop-portal` 等跨进程服务走会话总线。
- **配置热重载**：组件监听 `SIGHUP` 或 `inoxctl reload`，重新读取 TOML 并原子应用；重载失败保留旧配置并记录警告。

## 当前实现边界

架构图描述的是目标形态。就本工作区当前状态而言：L2 的 `inox-compositor` 在 headless 路径上实现了输出创建、输入事件与多窗口平铺（见[里程碑与路线图](../roadmap.md)）；`inox-common` 已发布 `logging` 模块（XDG 路径解析（`paths`）计划随 `v0.2.0` 发布、尚未打 tag）；L3、L4 的组件仓库目前都还是骨架，合成器与 Shell 之间的数据流尚未真实联通。
