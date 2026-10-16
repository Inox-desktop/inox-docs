# 依赖管理

Inox 是多仓库项目：每个仓库独立版本、独立 CI，跨仓库依赖通过**锁定 tag 的 Git 依赖**管理。规则见根级 `AGENTS.md` §8 与 PRD §3.2。

## 跨仓库依赖

跨仓库引用 Inox crate 时，必须同时锁定 tag（或 rev）并声明显式版本：

```toml
[dependencies]
inox-config = { git = "https://github.com/Inox-desktop/inox-config", tag = "v0.1.0", version = "0.1" }
```

两条约束都有实际原因：

- **禁止未锁定的 `branch = "main"`**：上游 `main` 的任意一次提交都会改变下游的构建结果，CI 会因此随机失败，且失败与下游的改动无关。
- **必须同时声明 `version`**：`deny.toml` 中 `[bans] wildcards = "deny"` 会拒绝通配符依赖，而只有 `git` + `tag`、没有 `version` 的依赖会带一个隐式的 `*` 版本要求，从而被判为通配符依赖。`inox-compositor` 的 `Cargo.toml` 与 `AGENTS.md` 都记录了这个陷阱。

Git 依赖的来源还必须登记在消费仓库 `deny.toml` 的 `[sources] allow-git` 中，否则 `cargo deny check` 会失败 —— 这保证了「哪个仓库从哪个来源拉代码」是显式声明、可审查的。

## 升级顺序

跨仓库升级必须按固定顺序，避免下游在上下游版本不一致的状态下合并：

1. 在上游仓库实现改动，**发布新的 tag**（`vX.Y.Z`）；
2. 更新下游仓库的 Git 依赖到该 tag，跑通下游 CI；
3. 最后在 `inox-integration` 上验证组合兼容性，并把版本登记到元仓库的 [`compat.toml`](https://github.com/Inox-desktop/inox/blob/main/compat.toml)。

不要把「上游未发布 tag 的改动」以 `branch` 或 `rev` 的形式临时接入下游。若确有必要用 `rev`，它同样需要显式 `version`。

**不要在已发布的 tag 上重写历史。** 下游按 tag 锁定，tag 内容一旦改变，所有下游的构建结果都会与审查时的状态不一致。

## 组件兼容矩阵

`compat.toml` 位于元仓库根目录，由 `inox-integration` 消费，登记各组件的当前兼容版本：

```toml
[meta]
version = "0.1.0"
updated = "2026-10-16"

[components]
inox-common = "v0.1.0"
inox-compositor = "v0.1.0"
```

组件发布后在 `[components]` 段登记 `<name> = "<tag>"`。也就是说：**发布 tag 与登记矩阵是两个动作**，只做前者会让集成测试对不上版本。

## 新增第三方依赖

- 新增依赖需在 PR 描述中说明用途与许可证。
- 优先选择生态中被广泛使用的 crate。
- 只在**首个真实需要它的时刻**引入，不为「以后可能要用」提前声明抽象或依赖（ADR 0004 的排期原则）。
- 不随意升级 major 版本；升级需跑通 CI 并说明破坏性影响。

### 许可证白名单

仅允许 BSD / MIT / Apache-2.0 / ISC / Zlib 等宽松许可证；**禁止 GPL / AGPL / SSPL 进入核心依赖**。由 `cargo deny check` 强制，同时检查 RUSTSEC 漏洞与重复版本。

## MSRV

`rust-version` 目前不一致，是已知且被记录的状态：

| 仓库 | `rust-version` | 原因 |
|------|----------------|------|
| `inox-compositor` | `1.86` | `smithay` 0.7 依赖的 `calloop` 0.14.5 要求 rustc ≥ 1.86（ADR 0005） |
| `inox-common`、`inox-config` 等 L1 | `1.85` | 尚无拉高 MSRV 的依赖 |

根级 `AGENTS.md` §8 要求升级 MSRV 时说明破坏性影响；`inox-compositor` 的仓库级 `AGENTS.md` 也明确了这一差异需要在 L1 的升级节奏上保持关注。

## 本仓库（`inox-docs`）

`inox-docs` **没有第三方依赖**：它不含 Rust 代码、没有 `Cargo.toml`，只使用 mdBook 构建文档，因此不参与依赖图与 `cargo-deny` 检查，也不引入任何 mdBook 插件。
