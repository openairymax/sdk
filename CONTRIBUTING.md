# 贡献指南 (Contributing Guide) — SDK 管理仓

**最后更新**: 2026-08-04

本仓库（`sdk`）是 SDK 管理仓，聚合 6 个叶子仓库：`sdk-python`、`sdk-go`、
`sdk-rust`、`sdk-typescript`、`cli`、`tui`。实际代码开发在叶子仓库中进行；
本仓只承载文档、submodule 配置与许可。

## 1. 提交前检查

在任意叶子仓库提交前，请确保：

| 仓库 | 测试命令 |
|------|----------|
| sdk-python | `python -m pip install -e .[test] && pytest` |
| sdk-go | `go test ./...`（在 `sdk-go/` 模块根目录） |
| sdk-rust | `cargo test` |
| sdk-typescript | `npm ci && npm test` |
| cli | `cargo test` |
| tui | `cargo test` |

CI（各仓库 `.github/workflows/ci.yml`）会真实运行上述测试，失败即任务失败。

## 2. 版本策略

**0.1.1 是唯一奠基版本**。任何 SDK 组件的版本声明（Cargo.toml / pyproject.toml /
package.json / go 版本常量 / `__version__` 等）必须与当前版本一致，不允许引入
0.1.2 / 0.2.0 / 1.0.0 等中间版本。新增组件默认从 0.1.1 开始。

## 3. 许可与署名

所有叶子仓库均为 **AGPL-3.0-or-later OR Apache-2.0** 双许可（SPDX:
`AGPL-3.0-or-later OR Apache-2.0`）。每个源文件必须保留 SPDX 头；每个提交必须
包含 `Signed-off-by:`（DCO）。

## 4. 提交信息

遵循内核风格：`subsystem: summary` + 正文（why > how > impact）+ tags。

---

© 2025-2026 SPHARX Ltd. 保留所有权利。
