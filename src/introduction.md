# 简介

Inox 是一个用 Rust（stable，2024 edition）编写、基于 Smithay 与 Iced 的模块化 Wayland 桌面环境，采用 AI-first 方式开发，CI（GitHub Actions）是唯一的构建与测试验证途径。

本仓库 `inox-docs` 汇总 Inox 的架构说明、组件职责与开发约定，作为各组件仓库之外的统一文档入口。

文档范围包括：分层架构与依赖方向、各组件仓库说明、配置与 IPC 约定、构建与 CI 流程，以及面向使用者的操作说明。

产品需求以元仓库的 [PRD](https://github.com/Inox-desktop/inox/blob/main/PRD.md) 为准，架构决策记录见元仓库 `docs/adr/`，所有贡献者与 AI 代理必须遵守的硬性规则见根级 [AGENTS.md](https://github.com/Inox-desktop/inox/blob/main/AGENTS.md)。

文档站点使用 mdBook 构建，CI 通过 `mdbook build` 验证；内容从 M0 骨架开始，随后续里程碑逐步补充。
