# Airymax SDK — 多语言开发者工具包

> Airymax AI 智能体运行时平台的开发者工具包管理仓。
> [airymaxhub](https://atomgit.com/openairymax/airymaxhub) 伞仓下五个管理仓之一。

**语言:** [English](README.md) | 简体中文

[![Version](https://img.shields.io/badge/version-0.1.1-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python->=3.8-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## 概述

**`sdk` 管理仓**是 Airymax 平台面向开发者的打包层。它以 git submodule 形式聚合 **6 个叶子仓**，为 Airymax AI 智能体运行时提供统一一致的开发者接口：

- **4 语言 SDK** — Python、Go、Rust、TypeScript
- **2 个交互式工具** — `cli`（命令行工具）和 `tui`（终端 UI 工具）

四个 SDK 采用同一套架构：每个语言 SDK 暴露一个 HTTP 客户端层（`Client` / `APIClient`），以及四个业务模块管理器 —— `TaskManager`（任务）、`MemoryManager`（记忆）、`SessionManager`（会话）与 `SkillManager`（技能）。基于这些 SDK 构建的智能体应用是**运行时租户** — 通过 HTTP / JSON-RPC 2.0 经 SDK 调用平台能力，而非直接接触内核内部。

本管理仓只承载文档、submodule 接线与许可证，所有实现均位于叶子仓中。

## 仓库结构

```
sdk/                       # 管理仓（本仓）
├── sdk-python/            # Python SDK 叶子仓（submodule）
├── sdk-go/                # Go SDK 叶子仓（submodule）
├── sdk-rust/              # Rust SDK 叶子仓（submodule）
├── sdk-typescript/        # TypeScript SDK 叶子仓（submodule）
├── cli/                   # cli 叶子仓（submodule，目录名：cli/）
├── tui/                   # tui 叶子仓（submodule，目录名：tui/）
├── .gitmodules            # submodule 定义
├── LICENSE                # AGPL-3.0 + Apache-2.0 双许可证全文
├── NOTICE                 # 版权、商标与第三方声明
├── README.md              # 英文版
└── README_zh.md           # 本文件（中文版）
```

## 叶子仓

| 模块 | 目录 | 仓库 URL | 语言 | 说明 |
|------|------|----------|------|------|
| **sdk-python** | `sdk-python/` | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK（`agentrt` 包，3.8+） |
| **sdk-go** | `sdk-go/` | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK（模块 `github.com/spharx/agentrt/sdk/go/agentrt`，Go 1.22+） |
| **sdk-rust** | `sdk-rust/` | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK（crate `agentrt-rs`，edition 2021） |
| **sdk-typescript** | `sdk-typescript/` | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK（npm 包 `@agentrt/sdk`，TS 5.0+） |
| **cli** | `cli/` | `git@atomgit.com:openairymax/cli.git` | Rust | 命令行工具，用于运行时运维 |
| **tui** | `tui/` | `git@atomgit.com:openairymax/tui.git` | Rust | 终端 UI 工具，用于交互式智能体会话 |

> 注意：`cli` 与 `tui` 模块的目录名与仓库名一致 —— 这两个交互式工具不使用 `sdk-` 前缀。

## SDK 架构

每个 SDK 都是 AgentRT 运行时的纯 HTTP 客户端，**不通过原生 FFI 绑定 Core C ABI** —— SDK 直接经 HTTP / JSON-RPC 2.0 与 Gateway 通信。每个语言 SDK 暴露一个底层 `APIClient` / `Client` 用于原始请求，其上封装四个符合语言习惯的业务模块管理器。

```
┌──────────────────────────────────────────────────────────────────┐
│  业务模块管理器（每语言 4 个）                                      │
│  TaskManager · MemoryManager · SessionManager · SkillManager      │
├──────────────────────────────────────────────────────────────────┤
│  HTTP 客户端层（APIClient / Client）                               │
│  agentrt (Python) · agentrt (Go) · agentrt-rs (Rust) ·            │
│  @agentrt/sdk (TypeScript)                                        │
├──────────────────────────────────────────────────────────────────┤
│  传输层：HTTP / JSON-RPC 2.0（无原生 FFI / Core C ABI）            │
└──────────────────────────────────────────────────────────────────┘
        │
        ▼   HTTP / JSON-RPC 2.0
┌──────────────────────────────────────────────────────────────────┐
│  AgentRT 运行时（gateway_d）— 内核服务与资源总线                    │
└──────────────────────────────────────────────────────────────────┘
```

### 上游依赖

- **运行时** — 通过 HTTP / JSON-RPC 2.0 连接到运行中的 AgentRT 实例（`gateway_d`）。
- **协议** — 使用 `protocols/` 管理仓中定义的 AgentsIPC 协议。
- **配置** — 运行时端点与凭据由 `ecosystem/manager/` 管理。

### 下游消费者

- **智能体应用** — 用户编写的智能体导入语言 SDK。
- **`cli` / `tui`** — 独立 Rust 工具，经 HTTP 与 Gateway 通信（不链接语言 SDK）。
- **参考示例** — `ecosystem/examples/` 下的智能体示例。

## 模块管理器 API

每个语言 SDK 暴露一个 HTTP 客户端与四个业务模块管理器，四种语言的管理器 API 对齐一致。

| 管理器 | 资源层 | 职责 |
|--------|--------|------|
| `TaskManager` | 任务 | 任务提交、查询、等待、取消、列表、批量操作 |
| `MemoryManager` | 记忆 | 分层记忆写入 / 读取 / 检索 |
| `SessionManager` | 会话 | 会话生命周期管理 |
| `SkillManager` | 技能 | 技能注册、调用、管理 |

```
Client（HTTP 层）
├── TaskManager    →  submit / get / wait / cancel / list
├── MemoryManager  →  write / read / search
├── SessionManager →  create / get / delete
└── SkillManager   →  load / invoke / list
```

## 构建与安装

### Python（sdk-python）

```bash
pip install agentrt
```

### Go（sdk-go）

```bash
go get github.com/spharx/agentrt/sdk/go/agentrt
```

### Rust（sdk-rust）

```bash
cargo add agentrt-rs
```

### TypeScript（sdk-typescript）

```bash
npm install @agentrt/sdk
# 或: pnpm add @agentrt/sdk / yarn add @agentrt/sdk
```

### CLI 与 TUI（cli / tui）

两个工具都是 Rust 二进制，随 `cli` 与 `tui` 叶子仓分发。从源码构建：

```bash
# 在 cli/ submodule 目录内
cargo install --path .

# 在 tui/ submodule 目录内
cargo install --path .
```

> **边界约定（SSoT）**：`cli` / `tui` 是面向开发者的独立 Rust 交互工具，与
> 核心仓内置的 C 工具 `agentrt/tools/airy_cli`（随运行时源码分发）定位互补：
> - `agentrt/tools/airy_cli`（C）— 运行时自带交互入口，**进程内直接链接
>   内核库**（GCCP / 认知管线），零外部依赖即用；内置交互 chat 与全屏 TUI
> - `sdk/cli`（Rust）— 独立 HTTP 租户，脚手架 / 配置 / 市场 / 部署类运维命令
> - `sdk/tui`（Rust）— 独立 HTTP 租户，可视化多面板交互终端界面
> 通信通道：`sdk/cli` / `sdk/tui` 经 Gateway HTTP（JSON-RPC 2.0，默认
> `http://localhost:8080`）与运行时通信；`airy_cli` 不依赖 HTTP，直连内核。
> 运行时互切：`sdk/tui` 内 F8 切换 exec `airy_cli`；`airy_cli` 内 `/tui`
> 命令 exec `agentrt-tui`（两二进制同装于 `$AIRY_HOME/bin`）。三者版本号
> 统一跟随 0.1.5。

## 快速入门

所有示例假定本地已运行 AgentRT 实例，地址为 `http://localhost:8080`。

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://localhost:8080")

# Task: submit a task
task = client.submit_task("analyze quarterly metrics")

# Memory: write a memory record
memory_id = client.write_memory("quarterly metrics", metadata={"tag": "report"})
```

`agentrt.modules` 包还暴露了模块管理器（`TaskManager`、`MemoryManager`、`SessionManager`、`SkillManager`），用于类型化访问。

### Go

```go
package main

import (
    "context"
    "fmt"

    "github.com/spharx/agentrt/sdk/go/agentrt"
    "github.com/spharx/agentrt/sdk/go/agentrt/client"
    "github.com/spharx/agentrt/sdk/go/agentrt/modules/task"
)

func main() {
    ctx := context.Background()
    c, err := client.NewClient(agentrt.WithEndpoint("http://localhost:8080"))
    if err != nil {
        panic(err)
    }

    tasks := task.NewTaskManager(c)
    t, err := tasks.Submit(ctx, "analyze quarterly metrics")
    if err != nil {
        panic(err)
    }
    fmt.Printf("%+v\n", t)
}
```

### Rust

```rust
use std::sync::Arc;

use agentrt_rs::client::Client;
use agentrt_rs::modules::task::TaskManager;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new("http://localhost:8080")?;
    let tasks = TaskManager::new(Arc::new(client));

    let task = tasks.submit("analyze quarterly metrics").await?;
    println!("{:?}", task);
    Ok(())
}
```

### TypeScript

```typescript
import { AgentRTClient, withEndpoint } from "@agentrt/sdk";

const client = new AgentRTClient(withEndpoint("http://localhost:8080"));

// Task manager: submit a task
const task = await client.tasks.submit("analyze quarterly metrics");
console.log(task);
```

## 分支策略

- **本管理仓** — 仅 `main` 分支，不在此创建功能分支。
- **叶子仓** — 活跃开发在 `feature/official-hubs-01` 分支进行；各叶子仓的 `main` 分支跟踪上一个稳定版本。

带 submodule 克隆本仓库：

```bash
git clone --recurse-submodules git@atomgit.com:openairymax/sdk.git
cd sdk
git submodule update --remote --checkout
```

## 许可证

采用 **AGPL v3 + Apache 2.0** 双许可证（SPDX: `AGPL-3.0-or-later OR Apache-2.0`），可任选其一。两份许可证全文详见 [LICENSE](LICENSE)，版权、商标与第三方声明详见 [NOTICE](NOTICE)。

### 双许可证使用指南

你可以**任选其一**适用——不是同时遵守两个，也不是都不遵守。

**SPDX 表达式**：`AGPL-3.0-or-later OR Apache-2.0`

| 你的场景 | 选择 | 原因 |
|----------|------|------|
| 构建**SaaS 网络服务**并修改 SDK | **AGPL v3** | 网络服务条款要求公开修改后的源代码 |
| 开发**开源 SDK 衍生作品**（copyleft 项目） | **AGPL v3** | 衍生作品必须同样以 AGPL 开源 |
| 在**商业闭源产品**中集成 SDK | **Apache 2.0** | 宽松许可证，允许闭源衍生 |
| 构建**企业内部工具** | **Apache 2.0** | 无需公开源代码 |
| 需要**专利保护** | **Apache 2.0** | 贡献者明确授予专利使用权 |
| 仅用于学习与研究 | **任一** | 两者均允许个人使用 |

权威许可证政策见 [12-license-policy.md](../docs/AirymaxOS/50-engineering-standards/12-license-policy.md)。

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
