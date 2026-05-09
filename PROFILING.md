# Performance Profiling

> **⚠️ For Development Only**: Profiling is intended for development and debugging. Do not enable profiling features in production deployments.

This guide explains how to profile Sv2 applications using [hotpath-rs](https://hotpath.rs/) to identify bottlenecks and optimize performance.

## Overview

Profiling is **zero-cost when disabled** - all instrumentation is gated behind feature flags and has no overhead unless explicitly enabled. For production deployments, always build without profiling features.

## Setup

### Building with Profiling

```bash
# Basic profiling (multi-threaded runtime)
cargo build --release --features hotpath

# Profiling with allocation tracking (single-threaded runtime)
cargo build --release --features hotpath-alloc
```

**Note**: The `hotpath-alloc` feature includes base profiling plus memory allocation tracking. It uses a single-threaded tokio runtime, which may affect performance characteristics.

## Profiling Modes

### Option 1: Static Report (Simple)

Prints a profiling summary when the application shuts down.

```bash
# Example with Pool
cd pool-apps/pool
cargo run --release --features hotpath -- -c config-examples/testnet4/pool-config-hosted-sv2-tp-example.toml
# ... run workload ...
# Press Ctrl+C to stop and view report
```

### Option 2: Live TUI Dashboard (Advanced)

Real-time monitoring with an interactive terminal dashboard.

**1. Install the hotpath CLI (once)**
```bash
cargo install hotpath --features='tui' --locked
```

**2. Start the dashboard**
```bash
hotpath console
```

**3. In another terminal, run your application**
```bash
# Example with Pool
cd pool-apps/pool
cargo run --release --features hotpath -- -c config-examples/testnet4/pool-config-hosted-sv2-tp-example.toml
```

The TUI will show real-time performance metrics as your application runs.

## Interpreting Results

Profiling data includes:
- **Call counts**: How many times each function was called
- **Total time**: Cumulative execution time
- **Average time**: Mean execution time per call
- **Percentage**: Time spent relative to total runtime
- **Percentiles**: p95, p99 timing statistics
- **Memory allocations**: Bytes allocated and allocation counts (with `--features hotpath-alloc`)

## Docker (hotpath-rs fork)

When running SRI apps via the hotpath-rs Docker Compose setup, each service exposes
both profiler and MCP endpoints on the host:

| Service | Profiler Port | MCP Port |
|---|---|---|
| pool_sv2 | 6781 | 6791 |
| jd_client_sv2 | 6782 | 6792 |
| translator_sv2 | 6783 | 6793 |

Install the TUI client and connect directly from the host:

```bash
cargo install hotpath --features='tui' --locked
hotpath console --metrics-host localhost --metrics-port 6781   # pool
hotpath console --metrics-host localhost --metrics-port 6782   # JDC
hotpath console --metrics-host localhost --metrics-port 6783   # translator
```

Profiler status and thread data are available over HTTP:

```bash
curl -s http://localhost:6781/profiler_status
curl -s http://localhost:6781/threads
```

MCP endpoints are available over HTTP at `/mcp`:

```bash
curl -i http://localhost:6791/mcp
curl -i http://localhost:6792/mcp
curl -i http://localhost:6793/mcp
```

Example Claude MCP registration:

```bash
claude mcp add --transport http hotpath-pool http://localhost:6791/mcp
```

Optional MCP authentication:

```bash
HOTPATH_MCP_AUTH_TOKEN=secret-token docker compose up -d pool_sv2
claude mcp add --transport http hotpath-pool http://localhost:6791/mcp --header "Authorization: secret-token"
```

Monitoring API security policy:

- `monitoring_address` is configured in each app TOML.
- Prefer `127.0.0.1:<port>` (default secure posture) or a dedicated WireGuard IP.
- Avoid `0.0.0.0` unless you explicitly intend public exposure.

To build with `hotpath-alloc` and keep MCP enabled, set the build arg:

```bash
docker compose build --build-arg HOTPATH_FEATURES=hotpath-alloc,hotpath-mcp pool_sv2
```
