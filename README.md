# Airymax SDK

> Airymax 多语言 SDK 管理仓库

## 概述

SDK 管理仓聚合 Airymax 的 4 语言 SDK（Python/Go/Rust/TypeScript）和 2 命令行工具（CLI/TUI），为开发者提供跨语言的统一 Agent 运行时接口。

## 子模块

| 模块 | 仓库 | 语言 | 说明 |
|------|------|------|------|
| sdk-python | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK（含官方 Hooks 包） |
| sdk-go | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK |
| sdk-rust | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK |
| sdk-typescript | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK |
| cli | `git@atomgit.com:openairymax/cli.git` | Rust | 命令行工具（裸名） |
| tui | `git@atomgit.com:openairymax/tui.git` | Rust | 终端 UI 工具（裸名） |

## SDK 双层 API 架构

每个 SDK 提供 4 个嵌套资源客户端（共 16 个嵌套客户端）：

```
AgentRTClient
├── CognitionClient   # 认知层：任务/循环/推理
├── SafetyClient      # 安全层：审计/沙箱/策略
├── ToolClient        # 工具层：注册/调用/编排
└── ChatClient        # 对话层：LLM 路由/会话/流式
```

## 快速入门

```python
# Python SDK 示例
from agentrt import AgentRT
from agentrt.hooks import SecurityReminderHook

client = AgentRT(endpoint="http://localhost:18789")
task = client.submit_task('{"input": "analyze data"}')
```

```go
// Go SDK 示例
import "agentrt/client"

client := client.NewAPIClient("http://localhost:18789")
task, _ := client.SubmitTask(ctx, taskInput)
```

## 仓库信息

- **仓库 URL**: `git@atomgit.com:openairymax/sdk.git`
- **归属组织**: openairymax
- **分支策略**: 仅 `main` 分支
- **许可证**: AGPL v3 + Apache 2.0 双许可证

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
SPDX-License-Identifier: AGPL-3.0-or-later OR Apache-2.0
