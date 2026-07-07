# Airymax SDK — Multi-Language Developer Toolkit

> Developer toolkit management repository for the Airymax AI Agent Runtime Platform.
> One of five management repositories under the [airymaxhub](https://atomgit.com/openairymax/airymaxhub) umbrella.

**Language:** English | [简体中文](README_zh.md)

[![Version](https://img.shields.io/badge/version-0.1.1-5a6b7e)](https://atomgit.com/openairymax/sdk)
[![License](https://img.shields.io/badge/license-AGPL--3.0+Apache--2.0-4a90d9)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Rust](https://img.shields.io/badge/Rust-stable-DEA584?logo=rust&logoColor=white)](https://www.rust-lang.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

---

## Overview

The **`sdk` management repository** is the developer-facing packaging layer of the Airymax platform. It aggregates **6 leaf repositories** as git submodules and exposes a single, coherent developer surface for the Airymax AI Agent Runtime:

- **4 language SDKs** — Python, Go, Rust, TypeScript
- **2 interactive tools** — `cli` (command-line interface) and `tui` (terminal UI)

All four SDKs implement the same **double-layer API architecture**: each language SDK exposes **4 nested resource clients** (`CognitionClient` / `SafetyClient` / `ToolClient` / `ChatClient`), yielding **4 languages × 4 nested clients = 16 nested clients** in total. Agent applications built on these SDKs are **runtime tenants** — they invoke platform capabilities through the SDK rather than touching kernel internals directly.

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
| **sdk-python** | `sdk-python/` | `git@atomgit.com:openairymax/sdk-python.git` | Python | Python SDK (`agentrt` package, 3.10+) |
| **sdk-go** | `sdk-go/` | `git@atomgit.com:openairymax/sdk-go.git` | Go | Go SDK (module `agentrt-sdk-go`, Go 1.22+) |
| **sdk-rust** | `sdk-rust/` | `git@atomgit.com:openairymax/sdk-rust.git` | Rust | Rust SDK (crate `agentrt-sdk`, edition 2021) |
| **sdk-typescript** | `sdk-typescript/` | `git@atomgit.com:openairymax/sdk-typescript.git` | TypeScript | TypeScript SDK (npm package `@spharx/agentrt-sdk`, TS 5.0+) |
| **cli** | `cli/` | `git@atomgit.com:openairymax/cli.git` | Rust | Command-line interface tool for runtime ops |
| **tui** | `tui/` | `git@atomgit.com:openairymax/tui.git` | Rust | Terminal UI tool for interactive agent sessions |

> Note: the `cli` and `tui` modules use the same name for both the directory and the repository — no `sdk-` prefix is applied to these two interactive tools.

## SDK Architecture

Airymax SDKs are built on a **double-layer API**. A single stable binary contract (the Core C ABI) is exposed through per-language FFI bindings, wrapped in idiomatic language SDKs, and finally partitioned into four nested resource clients. This keeps behavior identical across languages while letting each SDK feel native to its ecosystem.

```
┌──────────────────────────────────────────────────────────────────┐
│  Layer 2 — Nested Resource Clients (4 per language × 4 = 16)      │
│  CognitionClient · SafetyClient · ToolClient · ChatClient         │
├──────────────────────────────────────────────────────────────────┤
│  Layer 1 — Language SDKs (4 languages)                            │
│  agentrt (Python) · agentrt-sdk-go · agentrt-sdk (Rust) ·         │
│  @spharx/agentrt-sdk (TypeScript)                                 │
├──────────────────────────────────────────────────────────────────┤
│  FFI Bindings (per-language: PyO3 / cgo / bindgen / napi-rs)      │
├──────────────────────────────────────────────────────────────────┤
│  Core C ABI — stable binary contract (AgentsIPC protocol surface) │
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
- **`cli` / `tui`** — interactive tools that link the SDKs internally.
- **Reference examples** — agents under `ecosystem/examples/`.

## Nested Client API

Each language SDK exposes a single root client (`AgentRTClient`) with four nested resource clients. The nesting is the second layer of the double-layer API and is identical across all four languages.

| Nested Client | Resource Layer | Responsibilities |
|---------------|----------------|------------------|
| `CognitionClient` | Cognition | Task submission, reasoning loops, time-sliced inference (`CoreLoopThree` / `TimeSliceInfer`) |
| `SafetyClient` | Safety | Policy audit, sandboxing, endogenous security (`Cupolas`) |
| `ToolClient` | Tool | Tool registration, invocation, multi-tool orchestration |
| `ChatClient` | Chat | LLM provider routing (`SystemicLLM`), session management, streaming responses |

```
AgentRTClient
├── cognition   →  CognitionClient
├── safety      →  SafetyClient
├── tool        →  ToolClient
└── chat        →  ChatClient
```

## Build & Install

### Python (sdk-python)

```bash
pip install agentrt
```

### Go (sdk-go)

```bash
go get github.com/spharx/agentrt-sdk-go
```

### Rust (sdk-rust)

```bash
cargo add agentrt-sdk
```

### TypeScript (sdk-typescript)

```bash
npm install @spharx/agentrt-sdk
# or: pnpm add @spharx/agentrt-sdk / yarn add @spharx/agentrt-sdk
```

### CLI & TUI (cli / tui)

Both tools are Rust binaries distributed as part of the `cli` and `tui` leaf repos. Build them from source:

```bash
# Inside the cli/ submodule directory
cargo install --path .

# Inside the tui/ submodule directory
cargo install --path .
```

## Quick Start

All examples assume a running AgentRT instance at `http://localhost:18789`.

### Python

```python
from agentrt import AgentRT

client = AgentRT(endpoint="http://localhost:18789")

# Cognition layer: submit a task
task = client.cognition.submit_task({"input": "analyze quarterly metrics"})
print(task.result)

# Safety layer: audit a tool call before execution
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

// Chat layer: streaming LLM response
const stream = await client.chat.stream({ prompt: "Summarize the latest agent run" });
for await (const chunk of stream) {
    process.stdout.write(chunk.delta);
}
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

Copyright (c) 2025-2026 SPHARX Ltd. All Rights Reserved.
