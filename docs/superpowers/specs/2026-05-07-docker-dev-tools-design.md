# Docker LXC Development Tools Design

## Goal

Extend the existing Docker LXC script so users can optionally add a modern Node.js and Python development environment during Docker container setup, without changing the default Docker-only behavior.

## Scope

Modify the existing Docker script pair:

- `ct/docker.sh`
- `install/docker-install.sh`

The script will keep Docker, Compose, Buildx, Portainer, Portainer Agent, and Docker TCP socket behavior intact. The new development tooling will be optional during fresh installs and non-invasive during updates.

## User Flow

During a fresh Docker LXC install, after Docker is installed, the script asks:

`Would you like to add Node.js + Python development tools? <y/N>`

If the user answers yes, the installer adds:

- Node.js 22 via the existing `setup_nodejs` helper
- Global Node modules: `pnpm`, `yarn`
- uv via the existing `setup_uv` helper
- Python 3.13 managed by uv
- APT packages: `git`, `build-essential`, `python3`, `python3-pip`, `python3-venv`, `python3-dev`

An environment variable may also enable non-interactive installs:

- `INSTALL_DEV_TOOLS=yes`

## Update Behavior

The existing Docker update path remains the primary behavior. If development tooling is already detected, the update script refreshes the fixed toolchain:

- Node.js 22 and global `pnpm,yarn`
- uv and Python 3.13
- core APT development packages

If development tooling is not present, updates do not silently install it.

## Resource Defaults

The existing Docker LXC defaults are lightweight. Because development tooling and Docker image builds need more headroom, the Docker CT defaults should be raised modestly:

- Disk: 8 GB
- RAM: 4096 MB
- CPU: keep 2 cores

## Error Handling

Use the repo's existing helpers and conventions:

- `$STD` for silent or verbose command execution
- `msg_info`, `msg_ok`, `msg_error`
- `ensure_dependencies`
- `setup_nodejs`
- `setup_uv`

If development tooling installation fails, the script should fail clearly rather than reporting a complete development environment.

## Testing

Static checks should cover changed Bash files with `bash -n`. Full runtime verification requires a Proxmox host because the script depends on `pct`, LXC networking, systemd, APT repositories, and Docker daemon startup.
