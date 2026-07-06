# Airymax SDK — 多语言开发工具包

> Airymax AI 智能体运行时平台的统一 SDK 层。
> [airymaxhub](https://atomgit.com/openairymax/airymaxhub) 伞仓下四个管理仓之一。

**语言:** [English](README.md) | 简体中文

[![Version](https://img.shields.io/badge/version-0.1.1-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## 概述

**SDK 管理仓**聚合 Airymax 的 4 语言 SDK 和 2 个命令行工具，为开发者提供跨语言的统一 Agent 运行时接口。每个 SDK 实现相同的双层 API 架构，包含 4 个嵌套资源客户端（4 语言共 16 个嵌套客户端）。

使用这些 SDK 开发的 Agent 应用是**运行时租户** — 通过 SDK 调用系统能力，而非直接访问内核内部。

## 叶子仓

| 模块 | 仓库 | 语言 | 说明 |
|------|------|------|------|
| **sdk-python** | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK（含官方 Hooks 包：`agentrt.hooks`） |
| **sdk-go** | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK |
| **sdk-rust** | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK |
| **sdk-typescript** | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK（npm 包） |
| **cli** | `git@atomgit.com:openairymax/cli.git` | Rust | 命令行工具 |
| **tui** | `git@atomgit.com:openairymax/tui.git` | Rust | 终端 UI 工具 |

## 双层 API 架构

每个 SDK 提供 4 个嵌套资源客户端（4 语言共 16 个嵌套客户端）：

```
AgentRTClient
├── CognitionClient   # 认知层：任务 / 循环 / 推理
├── SafetyClient      # 安全层：审计 / 沙箱 / 策略
├── ToolClient        # 工具层：注册 / 调用 / 编排
└── ChatClient        # 对话层：LLM 路由 / 会话 / 流式
```

### 上游依赖

- **运行时**：通过 HTTP/JSON-RPC 2.0 连接到运行中的 AgentRT 实例（gateway_d）
- **协议**：使用 `protocols/` 中定义的 AgentsIPC 协议
- **配置**：由 `ecosystem/manager/` 管理

### 下游消费者

- **Agent 应用**：用户编写的 Agent 导入 SDK
- **CLI/TUI 工具**：交互式工具内部使用 SDK
- **示例**：`ecosystem/examples/` 中的参考 Agent

## 快速入门

### Python

```python
from agentrt import AgentRT
from agentrt.hooks import SecurityReminderHook

client = AgentRT(endpoint="http://localhost:18789")
client.register_hook(SecurityReminderHook())
task = client.cognition.submit_task('{"input": "analyze data"}')
print(task.result)
```

### Go

```go
import "agentrt/client"

client := client.NewAgentRTClient("http://localhost:18789")
task, err := client.Cognition.SubmitTask(ctx, taskInput)
```

### Rust

```rust
use agentrt_sdk::prelude::*;

let client = AgentRTClient::new("http://localhost:18789");
let task = client.cognition().submit_task(r#"{"input": "analyze data"}"#).await?;
```

### TypeScript

```typescript
import { AgentRTClient } from "@spharx/agentrt-sdk";

const client = new AgentRTClient({ endpoint: "http://localhost:18789" });
const task = await client.cognition.submitTask({ input: "analyze data" });
```

## 仓库结构

```
sdk/
├── sdk-python/       ← Python SDK + 官方 Hooks 包
├── sdk-go/           ← Go SDK
├── sdk-rust/         ← Rust SDK
├── sdk-typescript/   ← TypeScript SDK（npm）
├── cli/              ← CLI 工具（Rust）
├── tui/              ← 终端 UI（Rust）
├── .gitmodules       ← Submodule 定义
└── README.md         ← 本文件
```

## 分支策略

- 本管理仓：仅 **`main`** 分支
- 叶子仓：**`feature/official-hubs-01`**（活跃开发）

## 许可证

采用 **AGPL v3 + Apache 2.0** 双许可证（SPDX: `AGPL-3.0-or-later OR Apache-2.0`）。详见 [LICENSE](LICENSE)。

Copyright (c) 2025-2026 **SPHARX Ltd.** All Rights Reserved.
