# 里程碑与路线图

里程碑划分与验收标准（DoD）以 PRD §13 为准；当前状态以元仓库 [README 第 4 节](https://github.com/Inox-desktop/inox/blob/main/README.md)为准。两者不一致时以元仓库为准。

## 里程碑表

| 里程碑 | 目标 | 涉及仓库 | 验收标准（DoD） | 当前状态 |
|--------|------|----------|-----------------|----------|
| **M0** | 组织与仓库初始化、CI 模板、`AGENTS.md`、`LICENSE` | `.github`、所有仓库骨架 | 所有仓库含骨架、`AGENTS.md`、`LICENSE`、CI；空提交 CI 全绿 | **已完成** |
| **M1** | Headless 合成器：创建输出、渲染纯色、干净退出 | `inox-compositor` | CI 中 headless 创建输出并渲染帧，收到终止信号后退出码 0，clippy 无警告 | **已完成** |
| **M2** | 输入事件、多窗口、平铺布局 | `inox-compositor` | 处理键盘/指针事件；管理 ≥2 窗口；平铺布局通过 `proptest` | **已实现**（ADR 0008 仍为 `proposed`） |
| **M3** | XWayland 集成、剪贴板、拖放 | `inox-xwayland`、`inox-compositor` | headless 中启动并映射一个 X11 应用；剪贴板双向同步；拖放端到端通过 | 未开始 |
| **M4** | Iced Shell：面板、启动器、设置 | `inox-shell`、`inox-panel`、`inox-launcher`、`inox-settings` | 面板显示时钟/托盘；启动器可搜索并启动应用；设置读写并热重载；waydriver 截图验证通过 | 未开始 |
| **M5** | 会话管理、锁屏、通知、日常可用 | `inox-session`、`inox-lock`、`inox-notifications` | `inox-session` 拉起完整会话；锁屏可解锁；通知弹出与超时正确 | 未开始 |
| **M6** | 集成测试、性能优化、打包 | `inox-integration`、所有 | 端到端测试全绿；满足性能预算；产出至少 2 种分发物 | 未开始 |

## 当前状态说明

M0 与 M1 已完成，M2 已在 `inox-compositor` 实现：

- **M1**（[ADR 0005](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0005-headless-compositor-runtime.md)，状态 `accepted`）：自建 surfaceless EGL + `GlesRenderer` + 离屏渲染目标的 headless 渲染路径；创建 `wl_output`、渲染纯色帧、收到终止信号后退出码 0。
- **M2**（[ADR 0008](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0008-m2-input-windows-tiling.md)，状态 **`proposed`**）：在 M1 的 headless 路径上扩展，经 `wl_seat` 处理键盘与指针事件，以 `xdg-shell` toplevel 为窗口并由 Smithay `desktop::Space` 管理 ≥2 个窗口，按 master/stack 平铺（`proptest` 验证）。注意 ADR 本身尚未转为 `accepted` —— 实现已合入 `main`，决策记录仍是提案状态。

## 仍未完成的部分

以下能力在已完成的里程碑范围内**没有**被覆盖，属于后续里程碑或尚未排期：

- **真实输入设备链路**（libinput / udev / seat）：M2 的输入事件经进程内注入验证，真机输入风险推迟。
- **真实输出链路**（DRM/KMS）：合成器仍无 DRM 后端，多窗口平铺在离屏渲染上验证。
- **XWayland**：`inox-xwayland` 的 `main` 仍是 M0 骨架；以监管外部进程方式接入 XWayland 的工作在其特性分支上，尚未合入 `main`。
- **L3、L4 的全部组件**：`inox-shell`、`inox-panel`、`inox-launcher`、`inox-settings`、`inox-lock`、`inox-notifications`、`inox-session`、`inox-idle`、`inox-greet` 均为骨架。
- **配置读取**：`inox-config` 仍是骨架，合成器只解析命令行参数；配置文件格式是 ADR 0009 的提案。
- **`inox-protocols`**：仍是骨架，尚未定义任何协议绑定或 `inox-ipc`。

## 已发布的组件

元仓库 [compat.toml](https://github.com/Inox-desktop/inox/blob/main/compat.toml) 登记当前兼容版本：

| 组件 | 已登记版本 |
|------|------------|
| `inox-common` | `v0.1.0` |
| `inox-compositor` | `v0.1.0` |

`inox-common` 的 `paths` 模块（XDG 目录解析）计划随 `v0.2.0` 发布，尚未打 tag。
