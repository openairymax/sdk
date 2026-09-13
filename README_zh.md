# Airymax SDK — 多语言开发者工具包

> Airymax AI 智能体运行时（AgentRT）的开发者工具包 —— 命令行与终端 UI 工具，以及 Python、Go、Rust、TypeScript 语言绑定。
> 隶属 AtomGit 上的 [openairymax](https://atomgit.com/openairymax) 组织。

**语言：** [English](README.md) | 简体中文

[![Version](https://img.shields.io/badge/version-0.1.15-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python->=3.8-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## 概述

**`sdk` 仓库**是 Airymax 平台面向开发者的打包层。它以 git submodule 形式聚合 **6 个叶子仓**，为 AgentRT 运行时提供统一一致的开发者接口：

- **4 语言 SDK** — Python、Go、Rust、TypeScript
- **2 个交互式工具** — `cli`（命令行工具）和 `tui`（终端 UI 工具）

四个语言 SDK 采用同一套架构：HTTP 客户端层（`Client` / `APIClient`），其上封装四个业务模块管理器 —— `TaskManager`（任务）、`MemoryManager`（记忆）、`SessionManager`（会话）与 `SkillManager`（技能）。基于这些 SDK 构建的智能体应用是**运行时租户** —— 通过 HTTP / JSON-RPC 2.0 经 SDK 调用平台能力，而非直接接触内核内部。

本仓库承载文档、submodule 接线与许可证，所有实现均位于叶子仓中。

## 仓库结构

```
sdk/                       # 本仓库
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

| 模块 | 目录 | 仓库 | 语言 | 说明 |
|------|------|------|------|------|
| **sdk-python** | `sdk-python/` | [openairymax/sdk-python](https://atomgit.com/openairymax/sdk-python) | Python | Python SDK（`agentrt` 包，Python 3.8+） |
| **sdk-go** | `sdk-go/` | [openairymax/sdk-go](https://atomgit.com/openairymax/sdk-go) | Go | Go SDK（模块 `github.com/spharx/agentrt/sdk/go/agentrt`，Go 1.22+） |
| **sdk-rust** | `sdk-rust/` | [openairymax/sdk-rust](https://atomgit.com/openairymax/sdk-rust) | Rust | Rust SDK（crate `agentrt-rs`，edition 2021） |
| **sdk-typescript** | `sdk-typescript/` | [openairymax/sdk-typescript](https://atomgit.com/openairymax/sdk-typescript) | TypeScript | TypeScript SDK（npm 包 `@agentrt/sdk`，TypeScript 5.0+） |
| **cli** | `cli/` | [openairymax/cli](https://atomgit.com/openairymax/cli) | Rust | 命令行工具，用于运行时运维 |
| **tui** | `tui/` | [openairymax/tui](https://atomgit.com/openairymax/tui) | Rust | 终端 UI 工具，用于交互式智能体会话 |

> 注意：`cli` 与 `tui` 模块的目录名与仓库名一致 —— 这两个交互式工具不使用 `sdk-` 前缀。

## SDK 架构

每个语言 SDK 都是 AgentRT 运行时的纯 HTTP 客户端 —— 不通过原生 FFI 绑定 Core C ABI。每个 SDK 暴露一个底层 `APIClient` / `Client` 用于原始请求，其上封装四个符合语言习惯的业务模块管理器。

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

- **运行时** — 通过 HTTP / JSON-RPC 2.0 连接到运行中的 AgentRT 实例（`gateway_d`），`agent.run_stream` 走 SSE 事件流。

### 下游消费者

- **智能体应用** — 用户编写的智能体导入语言 SDK。
- **`cli` / `tui`** — 独立 Rust 工具，经 HTTP 与 Gateway 通信（不链接语言 SDK）。

## 模块管理器 API

四种语言的管理器 API 对齐一致：

| 管理器 | 资源 | 职责 |
|--------|------|------|
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

## 前置条件

SDK 与交互式工具都是运行中 AgentRT 运行时的客户端，请先安装运行时：

```bash
curl -fsSL "https://api.atomgit.com/api/v5/repos/openairymax/agentrt/contents/scripts/install.sh?ref=main" | python3 -c 'import json,sys,base64;sys.stdout.buffer.write(base64.b64decode(json.load(sys.stdin)["content"]))' | bash
```

（兼容入口 `https://atomgit.com/openairymax/agentrt/releases/download/latest/install.sh` 也可用。）

然后启动网关，例如 `airymaxrt start`。网关默认监听 `http://127.0.0.1:8080`；端口被占用时启动器会漂移到其他端口，并把实际端口写入 `$AIRY_HOME/run/gateway.port`（`$AIRY_HOME` 默认为 `~/.airymaxrt`）。

### 端点配置

每个 SDK 的端点解析顺序一致：显式选项 → `AGENTRT_ENDPOINT` 环境变量 → 内置默认值 `http://127.0.0.1:18789`。`cli` 工具使用 `--gateway-url` / `AGENTRT_GATEWAY_URL`（默认 `http://localhost:8080`）；`tui` 还会读取 `$AIRY_HOME/run/gateway.port`。

> 下文示例均显式传入端点（`http://127.0.0.1:8080`），可直接对接默认安装的运行时。若你的网关在其他地址，请相应调整取值或设置 `AGENTRT_ENDPOINT`。

## 安装

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

两个工具都是 Rust 二进制，从源码构建：

```bash
# 在 cli/ 目录内
cargo install --path .

# 在 tui/ 目录内
cargo install --path .
```

> **工具边界**：三个终端入口互补定位——
> - `agentrt/tools/airy_cli`（C，随运行时源码分发）— 运行时自带交互入口
> - `sdk/cli`（Rust）— 开发者运维命令行工具（脚手架 / 配置 / 市场 / 部署）
> - `sdk/tui`（Rust）— 开发者多面板可视化交互终端界面
> 三者统一经 gateway（JSON-RPC 2.0，默认 `http://127.0.0.1:8080`）与运行时
> 通信。`tui` crate 版本随运行时发布（0.1.15）；`cli` crate 独立版本化（0.1.8）。

## 快速入门

所有示例假定网关运行在 `http://127.0.0.1:8080`。

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://127.0.0.1:8080")

# 任务：提交任务
task = client.submit_task("analyze quarterly metrics")

# 记忆：写入记忆记录
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
    c, err := client.NewClient(agentrt.WithEndpoint("http://127.0.0.1:8080"))
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
    let client = Client::new("http://127.0.0.1:8080")?;
    let tasks = TaskManager::new(Arc::new(client));

    let task = tasks.submit("analyze quarterly metrics").await?;
    println!("{:?}", task);
    Ok(())
}
```

### TypeScript

```typescript
import { AgentRT } from "@agentrt/sdk";

const client = new AgentRT({ endpoint: "http://127.0.0.1:8080" });

// 任务管理器：提交任务
const task = await client.tasks.submit("analyze quarterly metrics");
console.log(task);
```

## 获取源码

聚合采用 git submodule，带全部叶子仓克隆本仓库：

```bash
git clone --recurse-submodules https://atomgit.com/openairymax/sdk.git
cd sdk
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

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
