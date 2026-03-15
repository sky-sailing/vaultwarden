# AGENTS.md

## Cursor Cloud specific instructions

### Overview

Vaultwarden is a Rust implementation of the Bitwarden server API. Single binary, no monorepo. Uses Rocket web framework and Diesel ORM with support for SQLite, MySQL, and PostgreSQL backends.

### Rocket Git Dependency Workaround

The Rocket dependency references a commit (`a298f844`) that is orphaned in the upstream repo (moved from `SergioBenitez/Rocket` to `rwf2/Rocket`). To build, you must:

1. Clone the Rocket repo and fetch the specific commit:
   ```
   git clone --bare https://github.com/rwf2/Rocket /tmp/rocket-bare
   git -C /tmp/rocket-bare fetch https://github.com/rwf2/Rocket a298f8446be64d58cbb7345b6baa026aa4b672b4
   git -C /tmp/rocket-bare update-ref refs/heads/vw-compat a298f8446be64d58cbb7345b6baa026aa4b672b4
   ```
2. Configure git to redirect the old URL to the local clone:
   ```
   git config --global url."/tmp/rocket-bare".insteadOf "https://github.com/SergioBenitez/Rocket"
   ```
3. Build with `CARGO_NET_GIT_FETCH_WITH_CLI=true`.

### Building and Running

- **Build (SQLite):** `CARGO_NET_GIT_FETCH_WITH_CLI=true cargo build --features sqlite`
- **Run (dev):** `WEB_VAULT_ENABLED=false ROCKET_PORT=8000 cargo run --features sqlite`
- **Lint:** `cargo fmt -- --check` and `CARGO_NET_GIT_FETCH_WITH_CLI=true cargo clippy --features sqlite -- -D warnings`
- **Test:** `CARGO_NET_GIT_FETCH_WITH_CLI=true cargo test --features sqlite`

### Non-Obvious Caveats

- `WEB_VAULT_ENABLED=false` must be set as an environment variable (not just in `.env`) or you must download the web vault files. The `.env` file setting may not override the default.
- The `data/` directory must exist before starting the server (`mkdir -p data`).
- Default dev port is 8000 (set via `ROCKET_PORT`). Production Docker images default to port 80.
- The `rust-toolchain` file pins Rust to 1.70.0; `rustup` will auto-select this version in the workspace.
- System libraries required: `libssl-dev`, `libpq-dev` (PostgreSQL), `libmariadb-dev` (MySQL). SQLite is bundled.
- Pre-commit hooks (`.pre-commit-config.yaml`) run `cargo fmt`, `cargo test`, and `cargo clippy` with all DB features enabled (`--features sqlite,mysql,postgresql,enable_mimalloc`).
