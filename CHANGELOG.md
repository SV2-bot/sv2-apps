# Changelog

## v0.4.0-hotpath-rs

Based on upstream [stratum-mining/sv2-apps v0.4.0](https://github.com/stratum-mining/sv2-apps/releases/tag/v0.4.0).

### Changes from upstream

- **Dockerfile**: Added `HOTPATH_FEATURES` build arg (defaults to `hotpath,hotpath-mcp`). All
  three `cargo build` commands use `--features ${HOTPATH_FEATURES}`.
- **Dockerfile**: Builder toolchain bumped to `rust:1.89-slim-bookworm` to satisfy
  `hotpath-mcp` dependency MSRV requirements.
- **docker-compose.yml**: Services build from the local Dockerfile instead of pulling
  pre-built images. Images tagged `:hotpath`. Pre-generated TOML configs mounted via
  `CONFIG_POOL`, `CONFIG_JDC`, `CONFIG_TPROXY` env vars. Healthchecks added for all
  services. Hotpath ports (6771/6772/6773 → 6770) exposed.
- **docker/README.md**: Replaced with pointer to `docker/AGENTS.md`.
- **docker/AGENTS.md**: Documents Dockerfile/compose changes, required env vars,
  branch/tag conventions, `HOTPATH_FEATURES` usage.
- **PROFILING.md**: Added Docker-specific section showing `hotpath console` usage
  with containerized services.
- **CHANGELOG.md**: This file.
