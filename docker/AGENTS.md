# Hotpath customizations

This is the hotpath-rs fork of sv2-apps. It builds locally-tagged Docker images
with `--features hotpath,hotpath-mcp` (or `--features hotpath-alloc,hotpath-mcp`), enabling extra hotpath-rs
protocol ports on each app.

## Dockerfile changes vs upstream

All three `cargo build` commands use `--features ${HOTPATH_FEATURES}` (cargo fetch does not accept --features):
- `pool_sv2`
- `jd_client_sv2`
- `translator_sv2`

`HOTPATH_FEATURES` defaults to `hotpath,hotpath-mcp`. Override with `--build-arg HOTPATH_FEATURES=hotpath-alloc,hotpath-mcp`
for memory allocation tracking with single-threaded tokio runtime.

## docker-compose.yml changes vs upstream

- Services build from the local Dockerfile instead of pulling pre-built images.
- Images are tagged `:hotpath` instead of upstream release tags.
- Config volumes mount pre-generated TOML configs via env vars (`CONFIG_POOL`,
  `CONFIG_JDC`, `CONFIG_TPROXY`) instead of template-based envsubst.
- Healthchecks added for all three services (curl monitoring API).
- Hotpath profiler and MCP ports are exposed via host networking with explicit env vars:

| Service | Profiler Port | MCP Port | Notes |
|---|---|---|---|
| pool_sv2 | 6781 | 6791 | `HOTPATH_METRICS_PORT=6781`, `HOTPATH_MCP_PORT=6791` |
| jd_client_sv2 | 6782 | 6792 | `HOTPATH_METRICS_PORT=6782`, `HOTPATH_MCP_PORT=6792` |
| translator_sv2 | 6783 | 6793 | `HOTPATH_METRICS_PORT=6783`, `HOTPATH_MCP_PORT=6793` |

All three services use `network_mode: host` because the hotpath crate's metrics server
binds to `127.0.0.1`, which is unreachable through Docker bridge port mapping.

Monitoring APIs (`9090`/`9091`/`9092`) are configured inside each app TOML file via
`monitoring_address`. Production deployments should bind monitoring to `127.0.0.1`
or a dedicated WireGuard IP, not `0.0.0.0`.

## Required environment variables

- `BITCOIN_SOCKET_PATH` — path to Bitcoin Core IPC socket (e.g. `/home/user/.sv2pi/bitcoin/data/node.sock`)
- `BITCOIN_IPC_DIR` — parent directory of the IPC socket (mounted to survive socket recreation)
- `CONFIG_POOL` — directory containing `pool-config.toml`
- `CONFIG_JDC` — directory containing `jdc-config.toml`
- `CONFIG_TPROXY` — directory containing `translator-config.toml`
- `DATA_POOL` — pool data directory
- `HOTPATH_MCP_PORT` — optional MCP port override per service
- `HOTPATH_MCP_AUTH_TOKEN` — optional MCP auth token (checked via Authorization header)

## Branch and tag convention

Hotpath releases follow `release/v{VERSION}-hotpath-rs` branches, tagged `v{VERSION}-hotpath-rs`,
branching from the upstream `release/v{VERSION}`.
