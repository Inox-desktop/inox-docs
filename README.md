# inox-docs

项目文档站点（mdBook）。属于 Inox 桌面环境的 **L1** 层。

本仓库是 Inox 在**各组件仓库之外**的统一文档入口：汇总架构说明、组件职责、配置参考与开发约定。站点的权威内容始终是元仓库的 [PRD](https://github.com/Inox-desktop/inox/blob/main/PRD.md)、[docs/adr](https://github.com/Inox-desktop/inox/tree/main/docs/adr) 与根级 [AGENTS.md](https://github.com/Inox-desktop/inox/blob/main/AGENTS.md)；本仓库不复制其正文，只做索引与解释。

## 状态

文档站点已按下列结构建立，正文用中文编写：

| 章节 | 内容 |
|------|------|
| 简介 | Inox 是什么、本仓库范围、如何阅读、权威来源 |
| 架构 | 分层与依赖方向、组件仓库清单与当前状态、协议矩阵、ADR 流程 |
| 配置 | 配置目录布局、`config.toml` / `compositor.toml` 参考、主题与设计令牌 |
| 开发 | 协作流程、持续集成、测试策略、依赖管理 |
| 使用 | 构建与运行 headless 合成器（并说明哪些能力尚不存在） |
| 附录 | 里程碑状态、术语表 |

关于配置的章节来自 [ADR 0009](https://github.com/Inox-desktop/inox/blob/main/docs/adr/0009-configuration-model-and-reload.md)（状态 `proposed`），页面开头均标注「提案，尚未发布」；描述组件现状时会明确区分「已实现」与「骨架」。

## 文档约定

- 正文用中文；命令、标识符、路径、TOML 键名与代码保持英文原样。
- 关于组件状态的描述必须可在仓库中核对，尚未实现的组件标注「骨架」「计划中」或「尚未实现」。
- 跨仓库引用使用绝对 URL；站内链接使用相对路径。

## 构建

站点使用 mdBook 构建，**不引入任何 mdBook 插件或额外工具链**：

```bash
mdbook build      # 生成静态站点到 book/
mdbook serve      # 本地预览
```

CI 在每次 push 与 PR 上执行 `mdbook build`，构建失败即为红。

## 开发约定

- 本项目遵循根级 [AGENTS.md](https://github.com/Inox-desktop/inox/blob/main/AGENTS.md) 与仓库级 [`AGENTS.md`](./AGENTS.md)。
- **CI（GitHub Actions）是唯一的构建与测试验证途径**，不在本地进行编译验证。
- 提交信息遵循 [Conventional Commits](https://www.conventionalcommits.org/)，scope 使用 `docs`。
- 修改章节后必须同步更新 `src/SUMMARY.md`，保证目录与实际的章节文件一致。

## 许可证

BSD 3-Clause，见 [`LICENSE`](./LICENSE)。
