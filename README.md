# Airymax SDK — Multi-Language Developer Toolkit

> Developer toolkit management repository for the Airymax AI Agent Runtime Platform.
> One of five management repositories under the [airymaxhub](https://atomgit.com/openairymax/airymaxhub) umbrella.

**Language:** English | [简体中文](README_zh.md)

[![Version](https://img.shields.io/badge/version-0.1.8-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python->=3.8-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## Overview

The **`sdk` management repository** is the developer-facing packaging layer of the Airymax platform. It aggregates **6 leaf repositories** as git submodules and exposes a single, coherent developer surface for the Airymax AI Agent Runtime:

- **4 language SDKs** — Python, Go, Rust, TypeScript
- **2 interactive tools** — `cli` (command-line interface) and `tui` (terminal UI)

All four SDKs share the same architecture: each language SDK exposes an HTTP client layer (`Client` / `APIClient`) plus four business module managers — `TaskManager` (tasks), `MemoryManager` (memory), `SessionManager` (sessions) and `SkillManager` (skills). Agent applications built on these SDKs are **runtime tenants** — they invoke platform capabilities through the SDK over HTTP / JSON-RPC 2.0 rather than touching kernel internals directly.

This management repo only carries documentation, submodule wiring, and licensing. All implementation lives in the leaf repositories.

## Repository Structure

```
sdk/                       # Management repository (this repo)
├── sdk-python/            # Python SDK leaf repo (submodule)
├── sdk-go/                # Go SDK leaf repo (submodule)
├── sdk-rust/              # Rust SDK leaf repo (submodule)
├── sdk-typescript/        # TypeScript SDK leaf repo (submodule)
├── cli/                   # cli leaf repo (submodule, directory name: cli/)
├── tui/                   # tui leaf repo (submodule, directory name: tui/)
├── .gitmodules            # Submodule definitions
├── LICENSE                # AGPL-3.0 + Apache-2.0 dual license full text
├── NOTICE                 # Copyright, trademark and third-party notices
├── README.md              # This file (English)
└── README_zh.md           # Chinese translation
```

## Leaf Repositories

| Module | Directory | Repository URL | Language | Description |
|--------|-----------|----------------|----------|-------------|
| **sdk-python** | `sdk-python/` | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK (`agentrt` package, 3.8+) |
| **sdk-go** | `sdk-go/` | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK (module `github.com/spharx/agentrt/sdk/go/agentrt`, Go 1.22+) |
| **sdk-rust** | `sdk-rust/` | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK (crate `agentrt-rs`, edition 2021) |
| **sdk-typescript** | `sdk-typescript/` | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK (npm package `@agentrt/sdk`, TS 5.0+) |
| **cli** | `cli/` | `git@atomgit.com:openairymax/cli.git` | Rust | Command-line interface tool for runtime ops |
| **tui** | `tui/` | `git@atomgit.com:openairymax/tui.git` | Rust | Terminal UI tool for interactive agent sessions |

> Note: the `cli` and `tui` modules use the same name for both the directory and the repository — no `sdk-` prefix is applied to these two interactive tools.

## SDK Architecture

Each SDK is a plain HTTP client for the AgentRT runtime. There is **no native FFI binding to a Core C ABI** — the SDKs talk to the Gateway directly over HTTP / JSON-RPC 2.0. Every language SDK exposes a low-level `APIClient` / `Client` for raw requests, with four idiomatic business module managers on top of it.

```
┌──────────────────────────────────────────────────────────────────┐
│  Business Module Managers (4 per language)                        │
│  TaskManager · MemoryManager · SessionManager · SkillManager      │
├──────────────────────────────────────────────────────────────────┤
│  HTTP Client Layer (APIClient / Client)                           │
│  agentrt (Python) · agentrt (Go) · agentrt-rs (Rust) ·            │
│  @agentrt/sdk (TypeScript)                                        │
├──────────────────────────────────────────────────────────────────┤
│  Transport: HTTP / JSON-RPC 2.0 (no native FFI / Core C ABI)      │
└──────────────────────────────────────────────────────────────────┘
        │
        ▼   HTTP / JSON-RPC 2.0
┌──────────────────────────────────────────────────────────────────┐
│  AgentRT Runtime (gateway_d) — kernel services & resource bus     │
└──────────────────────────────────────────────────────────────────┘
```

### Upstream Dependencies

- **Runtime** — connects to a running AgentRT instance (`gateway_d`) over HTTP / JSON-RPC 2.0.
- **Protocol** — speaks the AgentsIPC protocol defined in the `protocols/` management repo.
- **Configuration** — runtime endpoints and credentials are managed by `ecosystem/manager/`.

### Downstream Consumers

- **Agent applications** — user-written agents that import a language SDK.
- **`cli` / `tui`** — standalone Rust tools that talk to the Gateway over HTTP (they do not link the language SDKs).
- **Reference examples** — agents under `ecosystem/examples/`.

## Module Manager API

Each language SDK exposes an HTTP client plus four business module managers. The manager APIs are aligned across all four languages.

| Manager | Resource Layer | Responsibilities |
|---------|----------------|------------------|
| `TaskManager` | Tasks | Task submission, query, wait, cancel, list, batch operations |
| `MemoryManager` | Memory | Layered memory write / read / search |
| `SessionManager` | Sessions | Session lifecycle management |
| `SkillManager` | Skills | Skill registration, invocation, management |

```
Client (HTTP layer)
├── TaskManager    →  submit / get / wait / cancel / list
├── MemoryManager  →  write / read / search
├── SessionManager →  create / get / delete
└── SkillManager   →  load / invoke / list
```

## Build & Install

### Python (sdk-python)

```bash
pip install agentrt
```

### Go (sdk-go)

```bash
go get github.com/spharx/agentrt/sdk/go/agentrt
```

### Rust (sdk-rust)

```bash
cargo add agentrt-rs
```

### TypeScript (sdk-typescript)

```bash
npm install @agentrt/sdk
# or: pnpm add @agentrt/sdk / yarn add @agentrt/sdk
```

### CLI & TUI (cli / tui)

Both tools are Rust binaries distributed as part of the `cli` and `tui` leaf repos. Build them from source:

```bash
# Inside the cli/ submodule directory
cargo install --path .

# Inside the tui/ submodule directory
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
> 统一跟随 0.1.7。

## Quick Start

All examples assume a running AgentRT instance at `http://localhost:8080`.

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://localhost:8080")

# Task: submit a task
task = client.submit_task("analyze quarterly metrics")

# Memory: write a memory record
memory_id = client.write_memory("quarterly metrics", metadata={"tag": "report"})
```

The `agentrt.modules` package additionally exposes the module managers
(`TaskManager`, `MemoryManager`, `SessionManager`, `SkillManager`) for typed access.

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

## Branch Strategy

- **This management repo** — `main` only. No feature branches are created here.
- **Leaf repositories** — active development happens on `feature/official-hubs-01`. The `main` branch on each leaf repo tracks the last stable release.

When cloning this repo with submodules:

```bash
git clone --recurse-submodules git@atomgit.com:openairymax/sdk.git
cd sdk
git submodule update --remote --checkout
```

## License

Dual-licensed under **AGPL v3 + Apache 2.0** (SPDX: `AGPL-3.0-or-later OR Apache-2.0`). You may choose either license at your option. See [LICENSE](LICENSE) for the full text of both licenses and [NOTICE](NOTICE) for copyright, trademark and third-party notices.

### Dual License Guide

You may choose **either** license at your option — not both, not neither.

**SPDX Expression**: `AGPL-3.0-or-later OR Apache-2.0`

| If you are... | Choose | Why |
|---------------|--------|-----|
| Building a **SaaS** or network service that modifies the SDK | **AGPL v3** | Network service clause requires source disclosure |
| Developing **open-source** SDK derivatives (copyleft) | **AGPL v3** | Derivatives must remain open-source under AGPL |
| Using the SDK in **commercial closed-source** products | **Apache 2.0** | Permissive, allows proprietary derivatives |
| Building **enterprise internal tools** | **Apache 2.0** | No source disclosure required |
| Needing **patent protection** | **Apache 2.0** | Explicit patent grant from contributors |
| Just learning or researching | **Either** | Both permit personal use |

For the authoritative license policy, see [12-license-policy.md](../docs/AirymaxOS/50-engineering-standards/12-license-policy.md).

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
