# Airymax SDK — Multi-Language Development Kits

> Unified SDK layer for the Airymax AI Agent Runtime Platform.
> One of four management repositories under the [airymaxhub](https://atomgit.com/openairymax/airymaxhub) umbrella.

**Language:** English | [简体中文](README_zh.md)

[![Version](https://img.shields.io/badge/version-0.1.1-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## Overview

The **SDK management repo** aggregates Airymax's 4 language SDKs and 2 CLI tools, providing developers with a unified, cross-language Agent runtime interface. Each SDK implements the same double-layer API architecture with 4 nested resource clients (16 clients total across all languages).

Agent applications developed using these SDKs are **runtime tenants** — they invoke system capabilities through the SDK rather than directly accessing kernel internals.

## Leaf Repositories

| Module | Repository | Language | Description |
|--------|-----------|----------|-------------|
| **sdk-python** | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK (includes official Hooks package: `agentrt.hooks`) |
| **sdk-go** | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK |
| **sdk-rust** | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK |
| **sdk-typescript** | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK (npm package) |
| **cli** | `git@atomgit.com:openairymax/cli.git` | Rust | Command-line interface tool |
| **tui** | `git@atomgit.com:openairymax/tui.git` | Rust | Terminal UI tool |

## Double-Layer API Architecture

Each SDK provides 4 nested resource clients (16 nested clients total):

```
AgentRTClient
├── CognitionClient   # Cognition layer: tasks / loops / inference
├── SafetyClient      # Safety layer: audit / sandbox / policy
├── ToolClient        # Tool layer: register / invoke / orchestrate
└── ChatClient        # Chat layer: LLM routing / sessions / streaming
```

### Upstream Dependencies

- **Runtime**: Connects to a running AgentRT instance (gateway_d) via HTTP/JSON-RPC 2.0
- **Protocol**: Uses the AgentsIPC protocol defined in `protocols/`
- **Configuration**: Managed by `ecosystem/manager/`

### Downstream Consumers

- **Agent applications**: User-written agents that import the SDK
- **CLI/TUI tools**: Interactive tools that use the SDK internally
- **Examples**: Reference agents in `ecosystem/examples/`

## Quick Start

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

## Repository Structure

```
sdk/
├── sdk-python/       ← Python SDK + official Hooks package
├── sdk-go/           ← Go SDK
├── sdk-rust/         ← Rust SDK
├── sdk-typescript/   ← TypeScript SDK (npm)
├── cli/              ← CLI tool (Rust)
├── tui/              ← Terminal UI (Rust)
├── .gitmodules       ← Submodule definitions
└── README.md         ← This file
```

## Branch Strategy

- This management repo: **`main`** only
- Leaf repos: **`feature/official-hubs-01`** (active development)

## License

Dual-licensed under **AGPL v3 + Apache 2.0** (SPDX: `AGPL-3.0-or-later OR Apache-2.0`). See [LICENSE](LICENSE) for details.

Copyright (c) 2025-2026 **SPHARX Ltd.** All Rights Reserved.
