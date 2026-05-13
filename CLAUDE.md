# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**clash-for-linux-install** is a Bash-based one-click installer and management toolkit for running [mihomo](https://github.com/MetaCubeX/mihomo) or [clash](https://clash.wiki/) proxy kernels on Linux. It handles kernel download, service registration, subscription management, system proxy configuration, and Tun mode — all via shell scripts.

The project is entirely Bash (no compiled languages, no package manager, no build step). Fish shell support is provided via a wrapper in `scripts/cmd/clashctl.fish`.

## Commands

There is no build or test suite. The primary operations are:

```bash
# Install (interactive)
bash install.sh

# Install with arguments (kernel name, subscription URL)
bash install.sh mihomo https://example.com/sub

# Uninstall
bash uninstall.sh

# Lint (if shellcheck is available)
shellcheck install.sh uninstall.sh scripts/**/*.sh
```

After installation, the `clashctl` command and its aliases (`clashon`, `clashoff`, `clashui`, etc.) become available in the user's shell.

## Architecture

### Script layering

- **`.env`** — All configurable variables (kernel choice, install path, versions, GitHub proxy URL). Sourced by every script.
- **`scripts/cmd/common.sh`** — Shared constants (path variables, binary paths) and utility functions (port detection, YAML validation, config download, subconverter management, colored output helpers `_okcat`/`_failcat`/`_error_quit`). Sources `.env`.
- **`scripts/cmd/clashctl.sh`** — The main CLI. Defines all user-facing functions (`clashon`, `clashoff`, `clashctl`, `clashsub`, `clashtun`, `clashmixin`, etc.). Sources `common.sh`. Contains **placeholder tokens** (e.g., `placeholder_start`, `placeholder_stop`, `placeholder_status`) that `install.sh` replaces with init-system-specific commands via `sed`.
- **`scripts/preflight.sh`** — Installation logic: validation, argument parsing, binary download/extraction, init system detection, service installation, shell RC integration, and config merging. Sources `clashctl.sh`.
- **`install.sh`** / **`uninstall.sh`** — Entry points that orchestrate the above.

### Init system abstraction

`scripts/preflight.sh:_detect_init()` detects the system's init (systemd, SysVinit, OpenRC, runit) or falls back to `nohup` for containers/unprivileged users. Each init type defines arrays of commands (`service_start`, `service_stop`, `service_restart`, etc.) that get substituted into `clashctl.sh` via placeholder replacement at install time.

Init-specific service templates live in `scripts/init/`:
- `systemd.sh` — systemd unit file
- `SysVinit.sh` — SysV init script
- `OpenRC.sh` — OpenRC init script
- `runit.sh` — runit run script

### Config merge system

The core config workflow uses three YAML files under `resources/`:

1. **`config.yaml`** (`CLASH_CONFIG_BASE`) — The raw subscription config.
2. **`mixin.yaml`** (`CLASH_CONFIG_MIXIN`) — User customizations with highest priority. Supports `prefix`/`suffix`/`override`/`inject` operations on rules, proxies, and proxy-groups.
3. **`runtime.yaml`** (`CLASH_CONFIG_RUNTIME`) — The merged output loaded by the kernel.

`_merge_config()` in `clashctl.sh` uses `yq eval-all` to deep-merge these files. The `_custom` key in `mixin.yaml` is stripped before merging (used for app-level settings like `system-proxy.enable`).

### Subscription management

`clashsub` manages multiple subscriptions stored as individual YAML files in `resources/profiles/`, with metadata tracked in `resources/profiles.yaml`. The `subconverter` binary (downloaded at install time) handles format conversion when a subscription isn't natively compatible.

## Coding Conventions

- All scripts use `#!/usr/bin/env bash` and target Bash/Zsh (Fish has a separate wrapper).
- `.editorconfig`: LF line endings, 2-space indent.
- `.shellcheckrc` disables: SC1091 (can't follow source), SC2155 (declare and assign separately), SC2296, SC2153.
- Placeholder tokens in `clashctl.sh` and `common.sh` (e.g., `placeholder_start`, `placeholder_stop`) must not be removed — they are replaced by `preflight.sh:_install_service()` at install time.
- The `_is_regular_sudo` / `_is_root` distinction matters: regular sudo users get different service management behavior than root.
- Port conflict handling uses `_is_port_used()` + `_get_random_port()` — any new port-based feature should follow this pattern.
- Colored output uses `_okcat` (success), `_failcat` (warning), `_error_quit` (fatal). `_error_quit` calls `exec $SHELL -i` to exit.
