# Docker Development Tools Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend the existing Docker LXC script with optional Node.js 22 and Python 3.13 development tools.

**Architecture:** Keep the current Docker script pair and add a small optional development-tool block to the installer. Updates refresh the development toolchain only when it is already present, preserving Docker-only containers.

**Tech Stack:** Bash, Proxmox LXC helper functions, Docker Engine, NodeSource Node.js, uv-managed Python.

---

### Task 1: Update Docker CT Defaults

**Files:**
- Modify: `ct/docker.sh`

- [ ] **Step 1: Raise resource defaults**

Change the Docker LXC defaults from:

```bash
var_ram="${var_ram:-2048}"
var_disk="${var_disk:-4}"
```

to:

```bash
var_ram="${var_ram:-4096}"
var_disk="${var_disk:-8}"
```

- [ ] **Step 2: Run syntax check**

Run: `bash -n ct/docker.sh`

Expected: no output and exit code 0.

### Task 2: Add Optional Dev Tool Install Path

**Files:**
- Modify: `install/docker-install.sh`

- [ ] **Step 1: Add APT development dependencies after Docker install**

After Docker is installed and before Portainer prompts, add a conditional block controlled by either `INSTALL_DEV_TOOLS=yes` or a user prompt.

```bash
install_dev_tools="${INSTALL_DEV_TOOLS:-}"
if [[ -z "$install_dev_tools" ]]; then
  read -r -p "${TAB3}Would you like to add Node.js + Python development tools? <y/N> " prompt_dev
  if [[ ${prompt_dev,,} =~ ^(y|yes)$ ]]; then
    install_dev_tools="yes"
  fi
fi

if [[ ${install_dev_tools,,} =~ ^(y|yes|true|1)$ ]]; then
  msg_info "Installing Development Dependencies"
  ensure_dependencies \
    git \
    build-essential \
    python3 \
    python3-dev \
    python3-pip \
    python3-venv
  msg_ok "Installed Development Dependencies"

  NODE_VERSION="22" NODE_MODULE="pnpm,yarn" setup_nodejs
  PYTHON_VERSION="3.13" setup_uv
fi
```

- [ ] **Step 2: Run syntax check**

Run: `bash -n install/docker-install.sh`

Expected: no output and exit code 0.

### Task 3: Refresh Dev Tools During Updates

**Files:**
- Modify: `ct/docker.sh`

- [ ] **Step 1: Add conditional update block after Docker Engine update**

After the existing Docker Engine update block, add:

```bash
  if command -v node >/dev/null 2>&1 || command -v uv >/dev/null 2>&1; then
    msg_info "Updating Development Tools"
    ensure_dependencies \
      git \
      build-essential \
      python3 \
      python3-dev \
      python3-pip \
      python3-venv
    NODE_VERSION="22" NODE_MODULE="pnpm,yarn" setup_nodejs
    PYTHON_VERSION="3.13" setup_uv
    msg_ok "Updated Development Tools"
  fi
```

This keeps existing Docker-only LXC updates from silently installing Node.js or Python tooling.

- [ ] **Step 2: Run syntax check**

Run: `bash -n ct/docker.sh`

Expected: no output and exit code 0.

### Task 4: Final Verification

**Files:**
- Verify: `ct/docker.sh`
- Verify: `install/docker-install.sh`

- [ ] **Step 1: Run Bash syntax checks**

Run:

```bash
bash -n ct/docker.sh
bash -n install/docker-install.sh
```

Expected: both commands exit 0 with no output.

- [ ] **Step 2: Review diff**

Run: `git diff -- ct/docker.sh install/docker-install.sh`

Expected:

- Docker CT RAM default is 4096 MB.
- Docker CT disk default is 8 GB.
- Fresh installs ask before adding Node.js + Python tools.
- `INSTALL_DEV_TOOLS=yes` enables non-interactive dev tool installation.
- Updates refresh dev tools only if `node` or `uv` already exists.
