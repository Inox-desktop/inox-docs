# AGENTS.md — inox-docs

本仓库遵循根级 [AGENTS.md](https://github.com/Inox-desktop/inox/blob/main/AGENTS.md) 的**全部硬性规则**，本文件只补充仓库特有内容，不放宽任何根级约束。

## 职责

用户与开发者文档站点（mdBook）。所属层级：**L1**。

## 仓库特有约束

- 本仓库**不含 Rust 代码、无 `Cargo.toml`**：因此 `unsafe` / `unwrap()` / `tracing` 等 Rust 相关规则在本仓库不适用，本仓库也不参与依赖图与 `cargo-deny` 检查。
- 文档语言使用中文（面向用户的文档与 PRD 用中文）；命令、标识符与代码片段保持英文原样。
- 保持改动聚焦：只维护文档内容与 mdBook 配置，不引入构建脚本或额外工具链。
- 新增或修改文档后，必须保证 `mdbook build` 可通过（CI 以构建失败为红）。

## 主要依赖

无第三方依赖；仅使用 mdBook 构建文档。

## 命令

> 仅作参考，不作为本地验收依据；真正判定在 CI。

| 目的 | 命令 |
|------|------|
| 构建文档 | `mdbook build` |
| 本地预览 | `mdbook serve` |

## 测试要求

- 本仓库无单元测试要求，文档正确性由 CI 的 `mdbook build` 保证。
- 章节之间的链接必须有效，`src/SUMMARY.md` 必须与实际章节文件一致。

## 目录结构

```
src/
  SUMMARY.md          章节目录（必须与实际章节文件保持一致）
  introduction.md     简介与权威来源索引
  architecture/       分层、组件仓库、协议矩阵、ADR 流程
  configuration/      配置目录布局、合成器配置、主题与设计令牌
  development/        协作流程、CI、测试策略、依赖管理
  usage/              构建与运行
  roadmap.md          里程碑与当前状态
  glossary.md         术语表
book.toml       mdBook 配置
.github/        CI 工作流（mdbook build）
```

章节之间用相对路径互链；引用其他仓库时一律使用绝对 URL（`https://github.com/Inox-desktop/...`）。描述组件现状必须可在对应仓库中核对：尚未实现的组件要明确标注为「骨架」「计划中」或「尚未实现」。

## 提交信息格式

Conventional Commits，scope 使用 `docs`。
