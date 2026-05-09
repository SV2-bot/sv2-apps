# Frozen upstream config templates

These TOML templates are frozen copies from upstream `release/v0.4.0`.
The hotpath-rs deployment does not use them — it mounts pre-rendered configs
via `CONFIG_POOL`, `CONFIG_JDC`, `CONFIG_TPROXY` environment variables.

See `../AGENTS.md` for the hotpath deployment model.
