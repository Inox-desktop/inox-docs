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
src/            mdBook 源（SUMMARY.md 与各章节 .md）
book.toml       mdBook 配置
.github/        CI 工作流（mdbook build）
```

## 提交信息格式

Conventional Commits，scope 使用 `docs`。
