# 术语表

## 项目名词

| 术语 | 含义 |
|------|------|
| **Inox** | 本项目名，取自「不锈钢」（Inox），与 Rust 的「锈」形成反差 |
| **Compositor（合成器）** | Wayland 合成器，负责输入、渲染与窗口管理；本项目中即 `inox-compositor` |
| **Shell** | 桌面外壳，包含面板、启动器、通知等 UI；本项目中由 L3 的 Iced 组件构成 |
| **Session（会话）** | 负责拉起合成器、Shell 与系统服务的进程；本项目中即 `inox-session` |
| **Layer（层）** | 架构分层 L1–L4，见[总体架构](./architecture/overview.md) |
| **元仓库（meta repo）** | `inox` 仓库，只承载 PRD、ADR 与兼容矩阵，不含产品代码 |
| **ADR** | Architecture Decision Record，架构决策记录，见[架构决策记录](./architecture/decisions.md) |
| **DoD** | Definition of Done，里程碑的验收标准 |
| **compat.toml** | 元仓库中的组件兼容矩阵，登记各组件的兼容 tag |
| **`inox-comp`** | `inox-compositor` 的简写（PRD §3.1）；实际二进制名为 `inox-compositor` |
| **`inox-ipc`** | Inox 私有 Wayland 协议，定义于 `inox-protocols`，用于标准协议不覆盖的合成器–Shell 通信 |
| **`inoxctl`** | 规划中的统一命令行工具（`inox-cli` 仓库，尚未创建），也作为配置重载的触发入口 |

## 技术名词

| 术语 | 含义 |
|------|------|
| **Wayland** | 显示服务器协议；客户端通过套接字连接合成器，由合成器负责渲染 |
| **Smithay** | Rust 的 Wayland 合成器底层库；Inox 的合成器建立在它的抽象之上 |
| **calloop** | 基于回调的事件循环库；Inox 规定它作为主事件循环，禁止自建线程轮询循环 |
| **Iced** | Rust 的 GUI 工具包；Inox 的 Shell 组件基于它，遵循 Elm 架构（单向数据流） |
| **Elm 架构** | `Message` → `update` → `view` 的单向数据流模式 |
| **libseat** | 无 root 权限访问设备的会话库；合成器不以 root 运行 |
| **XWayland** | 让 X11 应用运行在 Wayland 会话中的兼容层；Inox 采用 `xwayland-satellite` |
| **rootless XWayland** | 每个 X11 客户端作为独立的 Wayland 窗口，而不是一整块根窗口 |
| **Layer shell（`wlr-layer-shell`）** | 用于面板、壁纸、锁屏等「层」表面的 Wayland 协议 |
| **`xdg-shell`** | 标准窗口协议；本项目中 toplevel 即一个窗口 |
| **Headless** | 无真实显示设备的运行/测试模式；输出是逻辑输出，渲染到离屏缓冲 |
| **llvmpipe** | Mesa 的软件光栅化器；CI 与无 GPU 环境用它渲染 |
| **surfaceless EGL** | 不依赖窗口系统或 DRM 设备的 EGL 平台（`EGL_MESA_platform_surfaceless`），M1 的 headless 路径基于它 |
| **Master/stack 平铺** | 平铺布局策略：master 列在工作区左侧，其余窗口在右侧堆叠 |
| **Design token（设计令牌）** | 主题中定义的颜色、排版、间距、圆角与动效时长；组件只消费令牌，禁止硬编码 |
| **Portal** | `xdg-desktop-portal`，桌面集成的标准接口层；Inox 计划由 `inox-portal` 提供后端 |
| **Fluent** | Project Fluent（`fluent-rs`），Inox 用于 UI 文案的国际化方案，禁止硬编码字符串 |
| **waydriver** | Wayland GUI 测试工具，用于在虚拟合成器中启动、截图并断言 Shell 组件 |
| **proptest** | Rust 的属性测试库；本项目中用于窗口管理与布局算法的不变量验证 |
| **Tracy** | 性能分析工具；用于帧时间与输入延迟的测量（PRD §8） |
| **cargo-deny** | 依赖检查工具，强制许可证白名单、RUSTSEC 漏洞与重复版本检查 |
| **XDG Base Directory** | 定义配置/数据/状态/缓存/运行时目录位置与回退规则的标准；Inox 在其下使用 `inox` 子目录 |
| **MSRV** | Minimum Supported Rust Version，最低支持的 Rust 版本 |
| **polyrepo** | 多仓库策略：每个组件一个独立仓库、独立版本、独立 CI |
| **`deny_unknown_fields`** | serde 选项，让未知字段成为解析错误而非静默忽略；Inox 的配置文件全部启用 |

## 相关页面

- 架构与分层：[总体架构](./architecture/overview.md)
- 配置相关名词的用法：[配置目录布局](./configuration/layout.md)、[主题与设计令牌](./configuration/theme.md)
