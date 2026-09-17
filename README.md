# Airymax SDK — Multi-Language Developer Toolkit

> Developer toolkit for the Airymax AI Agent Runtime (AgentRT) — console command
> surface and terminal-UI components, plus Python, Go, Rust and TypeScript bindings.
> Part of the [openairymax](https://atomgit.com/openairymax) organization on AtomGit.

**Language:** English | [简体中文](README_zh.md)

[![Version](https://img.shields.io/badge/version-0.1.15-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python->=3.8-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## Overview

The **`sdk` repository** is the developer-facing packaging layer of the Airymax platform. It aggregates **6 leaf repositories** as git submodules and exposes a single, coherent developer surface for the AgentRT runtime:

- **4 language SDKs** — Python, Go, Rust, TypeScript
- **2 interactive components** — `console` (console command surface library) and `tui` (terminal UI)

All four language SDKs share the same architecture: an HTTP client layer (`Client` / `APIClient`) topped by four business module managers — `TaskManager` (tasks), `MemoryManager` (memory), `SessionManager` (sessions) and `SkillManager` (skills). Agent applications built on these SDKs are **runtime tenants** — they invoke platform capabilities through the SDK over HTTP / JSON-RPC 2.0 rather than touching kernel internals directly.

This repository carries documentation, submodule wiring, and licensing. All implementation lives in the leaf repositories.

## Repository Structure

```
sdk/                       # This repository
├── sdk-python/            # Python SDK leaf repo (submodule)
├── sdk-go/                # Go SDK leaf repo (submodule)
├── sdk-rust/              # Rust SDK leaf repo (submodule)
├── sdk-typescript/        # TypeScript SDK leaf repo (submodule)
├── console/               # console leaf repo (submodule)
├── tui/                   # tui leaf repo (submodule)
├── .gitmodules            # Submodule definitions
├── LICENSE                # AGPL-3.0 + Apache-2.0 dual license full text
├── NOTICE                 # Copyright, trademark and third-party notices
├── README.md              # This file (English)
└── README_zh.md           # Chinese translation
```

## Leaf Repositories

| Module | Directory | Repository | Language | Description |
|--------|-----------|------------|----------|-------------|
| **sdk-python** | `sdk-python/` | [openairymax/sdk-python](https://atomgit.com/openairymax/sdk-python) | Python | Python SDK (`agentrt` package, Python 3.8+) |
| **sdk-go** | `sdk-go/` | [openairymax/sdk-go](https://atomgit.com/openairymax/sdk-go) | Go | Go SDK (module `github.com/spharx/agentrt/sdk/go/agentrt`, Go 1.22+) |
| **sdk-rust** | `sdk-rust/` | [openairymax/sdk-rust](https://atomgit.com/openairymax/sdk-rust) | Rust | Rust SDK (crate `agentrt-rs`, edition 2021) |
| **sdk-typescript** | `sdk-typescript/` | [openairymax/sdk-typescript](https://atomgit.com/openairymax/sdk-typescript) | TypeScript | TypeScript SDK (npm package `@agentrt/sdk`, TypeScript 5.0+) |
| **console** | `console/` | [openairymax/console](https://atomgit.com/openairymax/console) | Rust | Console command surface library (crate `agentrt-console`) |
| **tui** | `tui/` | [openairymax/tui](https://atomgit.com/openairymax/tui) | Rust | Terminal UI tool for interactive agent sessions |

> Note: the `console` and `tui` modules use the same name for both the directory and the repository — no `sdk-` prefix is applied to these two interactive components.

## SDK Architecture

Each language SDK is a plain HTTP client for the AgentRT runtime — there is no native FFI binding to a Core C ABI. Every SDK exposes a low-level `APIClient` / `Client` for raw requests, with four idiomatic business module managers on top of it.

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

- **Runtime** — connects to a running AgentRT instance (`gateway_d`) over HTTP / JSON-RPC 2.0, with SSE event streaming for `agent.run_stream`.

### Downstream Consumers

- **Agent applications** — user-written agents that import a language SDK.
- **`console` / `tui`** — Rust interactive components. `console` builds on `agentrt-rs` for transport and frame decoding; `tui` talks to the gateway over HTTP.

## Module Manager API

The manager APIs are aligned across all four languages:

| Manager | Resource | Responsibilities |
|---------|----------|------------------|
| `TaskManager` | Tasks | Task submit, query, wait, cancel, list, batch operations |
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

## Prerequisites

The SDKs and interactive tools are clients of a running AgentRT runtime. Install the runtime first:

```bash
curl -fsSL "https://api.atomgit.com/api/v5/repos/openairymax/agentrt/contents/scripts/install.sh?ref=main" | python3 -c 'import json,sys,base64;sys.stdout.buffer.write(base64.b64decode(json.load(sys.stdin)["content"]))' | bash
```

(A compatibility entry `https://atomgit.com/openairymax/agentrt/releases/download/latest/install.sh` is also available.)

Then start the gateway, e.g. `airymaxrt start`. By default the gateway listens on `http://127.0.0.1:8080`; if the port is taken, the launcher drifts to another port and records the actual one in `$AIRY_HOME/run/gateway.port` (`$AIRY_HOME` defaults to `~/.airymaxrt`).

### Endpoint Configuration

Every SDK resolves its endpoint in the same order: explicit option → `AGENTRT_ENDPOINT` environment variable → built-in default `http://127.0.0.1:18789`. The `tui` tool reads `$AIRY_HOME/run/gateway.port` in addition.

> The examples below pass the endpoint explicitly (`http://127.0.0.1:8080`) so they work against a default runtime install. If your gateway runs elsewhere, adjust the value or set `AGENTRT_ENDPOINT`.

## Installation

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

### Console & TUI (console / tui)

`console` is a Rust **library crate** (`agentrt-console`) — it defines no binary
of its own. It carries the console command surface (`commands/`) and the gateway
protocol client (`client.rs`), both built on the `agentrt-rs` transport. Consume it
as a path dependency:

```toml
[dependencies]
agentrt-console = { path = "sdk/console" }
```

`tui` is a Rust binary. Build it from source:

```bash
# Inside the tui/ directory
cargo install --path .
```

> **Tool boundary**: two terminal entry points play complementary roles —
> - `agentrt/tools/airy_cli` (C, ships with the runtime source) — the runtime's built-in interactive entry point
> - `sdk/console` (Rust, library) — the console command surface and gateway protocol client
> - `sdk/tui` (Rust, binary) — developer-oriented multi-panel interactive terminal UI
> All of them talk to the runtime through the gateway (JSON-RPC 2.0, default
> `http://127.0.0.1:8080`). The `tui` crate version follows the runtime release
> (0.1.16); the `console` crate version is maintained independently (0.1.16).

## Quick Start

All examples assume a running gateway at `http://127.0.0.1:8080`.

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://127.0.0.1:8080")

# Task: submit a task
task = client.submit_task("analyze quarterly metrics")

# Memory: write a memory record
memory_id = client.write_memory("quarterly metrics", metadata={"tag": "report"})
```

The `agentrt.modules` package additionally exposes the module managers (`TaskManager`, `MemoryManager`, `SessionManager`, `SkillManager`) for typed access.

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

// Task manager: submit a task
const task = await client.tasks.submit("analyze quarterly metrics");
console.log(task);
```

## Getting the Source

Aggregation uses git submodules. To clone this repository with all leaf repos:

```bash
git clone --recurse-submodules https://atomgit.com/openairymax/sdk.git
cd sdk
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

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
