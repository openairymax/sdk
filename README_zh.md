# Airymax SDK — 多语言开发者工具包

> Airymax AI 智能体运行时平台的开发者工具包管理仓。
> [airymaxhub](https://atomgit.com/openairymax/airymaxhub) 伞仓下五个管理仓之一。

**语言:** [English](README.md) | 简体中文

[![Version](https://img.shields.io/badge/version-0.1.1-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## 概述

**`sdk` 管理仓**是 Airymax 平台面向开发者的打包层。它以 git submodule 形式聚合 **6 个叶子仓**，为 Airymax AI 智能体运行时提供统一一致的开发者接口：

- **4 语言 SDK** — Python、Go、Rust、TypeScript
- **2 个交互式工具** — `cli`（命令行工具）和 `tui`（终端 UI 工具）

四个 SDK 实现同一套**双层 API 架构**：每个语言 SDK 暴露 **4 个嵌套资源客户端**（`CognitionClient` / `SafetyClient` / `ToolClient` / `ChatClient`），即 **4 语言 × 4 嵌套客户端 = 16 个嵌套客户端**。基于这些 SDK 构建的智能体应用是**运行时租户** — 通过 SDK 调用平台能力，而非直接接触内核内部。

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
| **sdk-python** | `sdk-python/` | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK（`agentrt` 包，3.10+） |
| **sdk-go** | `sdk-go/` | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK（模块 `agentrt-sdk-go`，Go 1.22+） |
| **sdk-rust** | `sdk-rust/` | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK（crate `agentrt-sdk`，edition 2021） |
| **sdk-typescript** | `sdk-typescript/` | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK（npm 包 `@spharx/agentrt-sdk`，TS 5.0+） |
| **cli** | `cli/` | `git@atomgit.com:openairymax/cli.git` | Rust | 命令行工具，用于运行时运维 |
| **tui** | `tui/` | `git@atomgit.com:openairymax/tui.git` | Rust | 终端 UI 工具，用于交互式智能体会话 |

> 注意：`cli` 与 `tui` 模块的目录名与仓库名一致 —— 这两个交互式工具不使用 `sdk-` 前缀。

## SDK 架构

Airymax SDK 基于**双层 API** 构建。一份稳定的二进制契约（Core C ABI）通过各语言的 FFI 绑定暴露，再封装为符合语言习惯的 SDK，最后划分为四个嵌套资源客户端。这样既保证跨语言行为一致，又让每个 SDK 在各自生态中显得原生自然。

```
┌──────────────────────────────────────────────────────────────────┐
│  第 2 层 — 嵌套资源客户端（每语言 4 个 × 4 语言 = 16 个）          │
│  CognitionClient · SafetyClient · ToolClient · ChatClient         │
├──────────────────────────────────────────────────────────────────┤
│  第 1 层 — 语言 SDK（4 种语言）                                    │
│  agentrt (Python) · agentrt-sdk-go · agentrt-sdk (Rust) ·         │
│  @spharx/agentrt-sdk (TypeScript)                                 │
├──────────────────────────────────────────────────────────────────┤
│  FFI 绑定（各语言：PyO3 / cgo / bindgen / napi-rs）                │
├──────────────────────────────────────────────────────────────────┤
│  Core C ABI — 稳定的二进制契约（AgentsIPC 协议接口）               │
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
- **`cli` / `tui`** — 交互式工具在内部链接 SDK。
- **参考示例** — `ecosystem/examples/` 下的智能体示例。

## 嵌套客户端 API

每个语言 SDK 暴露一个根客户端（`AgentRTClient`），其下挂四个嵌套资源客户端。这一嵌套结构是双层 API 的第二层，在四种语言中完全一致。

| 嵌套客户端 | 资源层 | 职责 |
|-----------|--------|------|
| `CognitionClient` | 认知 | 任务提交、推理循环、时间切片推理（`CoreLoopThree` / `TimeSliceInfer`） |
| `SafetyClient` | 安全 | 策略审计、沙箱、内生安全（`Cupolas`） |
| `ToolClient` | 工具 | 工具注册、调用、多工具编排 |
| `ChatClient` | 对话 | LLM 供应商路由（`SystemicLLM`）、会话管理、流式响应 |

```
AgentRTClient
├── cognition   →  CognitionClient
├── safety      →  SafetyClient
├── tool        →  ToolClient
└── chat        →  ChatClient
```

## 构建与安装

### Python（sdk-python）

```bash
pip install agentrt
```

### Go（sdk-go）

```bash
go get github.com/spharx/agentrt-sdk-go
```

### Rust（sdk-rust）

```bash
cargo add agentrt-sdk
```

### TypeScript（sdk-typescript）

```bash
npm install @spharx/agentrt-sdk
# 或: pnpm add @spharx/agentrt-sdk / yarn add @spharx/agentrt-sdk
```

### CLI 与 TUI（cli / tui）

两个工具都是 Rust 二进制，随 `cli` 与 `tui` 叶子仓分发。从源码构建：

```bash
# 在 cli/ submodule 目录内
cargo install --path .

# 在 tui/ submodule 目录内
cargo install --path .
```

## 快速入门

所有示例假定本地已运行 AgentRT 实例，地址为 `http://localhost:18789`。

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://localhost:18789")

# 认知层：提交任务
task = client.cognition.submit_task({"input": "analyze quarterly metrics"})
print(task.result)

# 安全层：在执行工具调用前进行审计
audit = client.safety.audit(tool="web_search", args={"q": "airymax"})
if audit.allowed:
    client.tool.invoke("web_search", {"q": "airymax"})
```

### Go

```go
package main

import (
    "context"
    "fmt"

    "github.com/spharx/agentrt-sdk-go/client"
)

func main() {
    c := client.NewAgentRTClient("http://localhost:18789")

    task, err := c.Cognition.SubmitTask(ctx, map[string]any{"input": "analyze quarterly metrics"})
    if err != nil {
        panic(err)
    }
    fmt.Println(task.Result)
}
```

### Rust

```rust
use agentrt_sdk::prelude::*;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = AgentRTClient::new("http://localhost:18789");

    let task = client
        .cognition()
        .submit_task(r#"{"input":"analyze quarterly metrics"}"#)
        .await?;

    println!("{}", task.result);
    Ok(())
}
```

### TypeScript

```typescript
import { AgentRTClient } from "@spharx/agentrt-sdk";

const client = new AgentRTClient({ endpoint: "http://localhost:18789" });

// 对话层：流式 LLM 响应
const stream = await client.chat.stream({ prompt: "Summarize the latest agent run" });
for await (const chunk of stream) {
    process.stdout.write(chunk.delta);
}
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
