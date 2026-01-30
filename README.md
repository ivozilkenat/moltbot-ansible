# Clawdbot Ansible Installer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Lint](https://github.com/ivozilkenat/moltbot-ansible/actions/workflows/lint.yml/badge.svg)](https://github.com/ivozilkenat/moltbot-ansible/actions/workflows/lint.yml)
[![Ansible](https://img.shields.io/badge/Ansible-2.14+-blue.svg)](https://www.ansible.com/)
[![Multi-OS](https://img.shields.io/badge/OS-Debian%20%7C%20Ubuntu%20%7C%20macOS-orange.svg)](https://www.debian.org/)

Automated, hardened installation of [Clawdbot](https://github.com/clawdbot/clawdbot) with Docker, Homebrew, and Tailscale VPN support for Linux and macOS.

## Features

- 🔒 **Firewall-first**: UFW (Linux) + Application Firewall (macOS) + Docker isolation
- 🔐 **Tailscale VPN**: Secure remote access without exposing services
- 🍺 **Homebrew**: Package manager for both Linux and macOS
- 🐳 **Docker**: Docker CE (Linux) / Docker Desktop (macOS)
- 🛡️ **Multi-OS Support**: Debian, Ubuntu, and macOS
- 🚀 **One-command install**: Complete setup in minutes
- 🔧 **Auto-configuration**: DBus, systemd, environment setup
- 📦 **pnpm installation**: Uses `pnpm install -g clawdbot@latest`

## Quick Start

### Automated Install (Recommended)

One-liner install with Tailscale auto-connect:

```bash
curl -fsSL https://raw.githubusercontent.com/ivozilkenat/moltbot-ansible/main/install.sh | bash -s -- \
  --ts-authkey YOUR_TAILSCALE_AUTHKEY \
  --ts-hostname YOUR_HOSTNAME
```

This will:
- Install all dependencies (Docker, Node.js, pnpm, etc.)
- Connect to Tailscale (using Headscale at vpn.ivo-zilkenat.de)
- Configure and start the Clawdbot gateway on port 18789

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--ts-authkey` | (optional) | Tailscale auth key - if not provided, Tailscale is installed but not connected |
| `--ts-hostname` | (optional) | Hostname for this machine in Tailnet |
| `--ts-login-server` | `https://vpn.ivo-zilkenat.de` | Custom login server URL |
| `--gateway-bind` | `lan` | Gateway bind mode: `lan`, `loopback`, `tailnet` |
| `--gateway-token` | (auto-generated) | Specific gateway token |

**Note:** If `--ts-authkey` is not provided, Tailscale will be installed but you must connect manually with:
```bash
sudo tailscale up --login-server https://vpn.ivo-zilkenat.de
```

### After Installation

```bash
sudo su - clawdbot
clawdbot onboard --no-install-daemon
```

### Basic Install (No Tailscale)

Install the latest stable version from npm without Tailscale auto-connect:

```bash
curl -fsSL https://raw.githubusercontent.com/ivozilkenat/moltbot-ansible/main/install.sh | sudo bash
```

### Development Mode

Install from source for development or testing:

```bash
# Clone the installer
git clone https://github.com/ivozilkenat/moltbot-ansible.git
cd moltbot-ansible

# Install in development mode
ansible-playbook playbook.yml --ask-become-pass -e clawdbot_install_mode=development
```

## What Gets Installed

- Tailscale (mesh VPN)
- UFW firewall (SSH + Tailscale ports only)
- Docker CE + Compose V2 (for sandboxes)
- Node.js 22.x + pnpm
- Clawdbot on host (not containerized)
- Systemd service (auto-start)

## Post-Install

After installation completes, the gateway is **automatically configured and running**.

The playbook:
- Deploys the config to `~/.clawdbot/clawdbot.json`
- Installs and starts the gateway daemon
- Generates an auth token (shown in the playbook output)

Switch to the clawdbot user to see the dashboard URL and token:

```bash
sudo su - clawdbot
```

The dashboard URL and auth token are displayed on login.

### Next Steps

1. **Open the dashboard** in your browser (URL shown on login)
2. **Paste the auth token** in dashboard settings
3. **Connect a messaging provider**:

```bash
# Login to WhatsApp/Telegram/Signal
clawdbot providers login

# Check status
clawdbot status
clawdbot logs
```

### Manual Setup (Alternative)

If you prefer manual configuration or need to reconfigure:

```bash
# Interactive wizard (overwrites existing config)
clawdbot onboard --install-daemon

# Or configure manually
clawdbot configure

# Manage gateway service
clawdbot gateway install
clawdbot gateway start
clawdbot gateway restart
clawdbot gateway stop
```

## Installation Modes

### Release Mode (Default)
- Installs via `pnpm install -g clawdbot@latest`
- Gets latest stable version from npm registry
- Automatic updates via `pnpm install -g clawdbot@latest`
- **Recommended for production**

### Development Mode
- Clones from `https://github.com/clawdbot/clawdbot.git`
- Builds from source with `pnpm build`
- Symlinks binary to `~/.local/bin/clawdbot`
- Adds helpful aliases:
  - `clawdbot-rebuild` - Rebuild after code changes
  - `clawdbot-dev` - Navigate to repo directory
  - `clawdbot-pull` - Pull, install deps, and rebuild
- **Recommended for development and testing**

Enable with: `-e clawdbot_install_mode=development`

## Security

- **Public ports**: SSH (22), Tailscale (41641/udp) only
- **Docker available**: For Clawdbot sandboxes (isolated execution)
- **Docker isolation**: Containers can't expose ports externally (DOCKER-USER chain)
- **Non-root**: Clawdbot runs as unprivileged user
- **Systemd hardening**: NoNewPrivileges, PrivateTmp

Verify: `nmap -p- YOUR_SERVER_IP` should show only port 22 open.

## Reverse Proxy Configuration

This installer configures Clawdbot to work behind a reverse proxy (like Traefik) by default.

### Default Network Access

The gateway is accessible from:
- **Tailscale**: `100.64.0.0/32` (your Traefik proxy host)
- **Local Network**: `192.168.1.0/24`

### Gateway Settings

By default, the gateway:
- Binds to `0.0.0.0:3000` (all interfaces)
- Trusts `X-Forwarded-*` headers from configured proxies
- UFW allows port 3000 from trusted networks only

### Customizing Network Access

Override in your vars.yml or command line:

```yaml
# Custom trusted networks (for UFW firewall rules)
clawdbot_allowed_networks:
  - { ip: "100.64.0.0/10", comment: "All Tailscale IPs" }
  - { ip: "10.0.0.0/8", comment: "Private network" }

# Custom trusted proxies for X-Forwarded headers
clawdbot_trusted_proxies:
  - "100.64.0.0/10"
  - "10.0.0.0/8"
```

### Proxy on Same Machine (localhost only)

If your reverse proxy runs on the **same host** as Clawdbot:

```yaml
# Only accept connections from localhost
clawdbot_gateway_bind: "127.0.0.1"
clawdbot_behind_proxy: true
clawdbot_trusted_proxies:
  - "127.0.0.1"
```

### No Proxy (direct access)

If not using a reverse proxy at all:

```yaml
clawdbot_gateway_bind: "0.0.0.0"
clawdbot_behind_proxy: false
```

### Traefik Example

Example Traefik configuration for routing to Clawdbot:

```yaml
http:
  routers:
    clawdbot:
      rule: "Host(`clawdbot.yourdomain.com`)"
      service: clawdbot
      tls:
        certResolver: letsencrypt
  services:
    clawdbot:
      loadBalancer:
        servers:
          - url: "http://clawdbot-host:3000"
```

## Documentation

- [Configuration Guide](docs/configuration.md) - All configuration options
- [Development Mode](docs/development-mode.md) - Build from source
- [Security Architecture](docs/security.md) - Security details
- [Technical Details](docs/architecture.md) - Architecture overview
- [Troubleshooting](docs/troubleshooting.md) - Common issues
- [Agent Guidelines](AGENTS.md) - AI agent instructions

## Requirements

### Linux (Debian/Ubuntu)
- Debian 11+ or Ubuntu 20.04+
- Root/sudo access
- Internet connection

### macOS
- macOS 11 (Big Sur) or later
- Homebrew will be installed automatically
- Admin/sudo access
- Internet connection

## What Gets Installed

### Common (All OS)
- Homebrew package manager
- Node.js 22.x + pnpm
- Clawdbot via `pnpm install -g clawdbot@latest`
- Essential development tools
- Git, zsh, oh-my-zsh

### Linux-Specific
- Docker CE + Compose V2
- UFW firewall (configured)
- Tailscale VPN
- systemd service

### macOS-Specific
- Docker Desktop (via Homebrew Cask)
- Application Firewall
- Tailscale app

## Manual Installation

### Release Mode (Default)

```bash
# Install dependencies
sudo apt update && sudo apt install -y ansible git

# Clone repository
git clone https://github.com/ivozilkenat/moltbot-ansible.git
cd moltbot-ansible

# Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# Run installation
./run-playbook.sh
```

### Development Mode

Build from source for development:

```bash
# Same as above, but with development mode flag
./run-playbook.sh -e clawdbot_install_mode=development

# Or directly:
ansible-playbook playbook.yml --ask-become-pass -e clawdbot_install_mode=development
```

This will:
- Clone clawdbot repo to `~/code/clawdbot`
- Run `pnpm install` and `pnpm build`
- Symlink binary to `~/.local/bin/clawdbot`
- Add development aliases to `.bashrc`

## Configuration Options

All configuration variables can be found in [`roles/clawdbot/defaults/main.yml`](roles/clawdbot/defaults/main.yml).

You can override them in three ways:

### 1. Via Command Line

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e clawdbot_install_mode=development \
  -e "clawdbot_ssh_keys=['ssh-ed25519 AAAAC3... user@host']"
```

### 2. Via Variables File

```bash
# Create vars.yml
cat > vars.yml << EOF
clawdbot_install_mode: development
clawdbot_ssh_keys:
  - "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGxxxxxxxx user@host"
  - "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... user@host"
clawdbot_repo_url: "https://github.com/YOUR_USERNAME/clawdbot.git"
clawdbot_repo_branch: "feature-branch"
tailscale_authkey: "tskey-auth-xxxxxxxxxxxxx"
EOF

# Use it
ansible-playbook playbook.yml --ask-become-pass -e @vars.yml
```

### 3. Edit Defaults Directly

Edit `roles/clawdbot/defaults/main.yml` before running the playbook.

### Available Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `clawdbot_user` | `clawdbot` | System user name |
| `clawdbot_home` | `/home/clawdbot` | User home directory |
| `clawdbot_install_mode` | `release` | `release` or `development` |
| `clawdbot_ssh_keys` | `[]` | List of SSH public keys |
| `clawdbot_repo_url` | `https://github.com/clawdbot/clawdbot.git` | Git repository (dev mode) |
| `clawdbot_repo_branch` | `main` | Git branch (dev mode) |
| `tailscale_authkey` | `""` | Tailscale auth key for auto-connect |
| `nodejs_version` | `22.x` | Node.js version to install |

See [`roles/clawdbot/defaults/main.yml`](roles/clawdbot/defaults/main.yml) for the complete list.

### Common Configuration Examples

#### SSH Keys for Remote Access

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e "clawdbot_ssh_keys=['ssh-ed25519 AAAAC3... user@host']"
```

#### Development Mode with Custom Repository

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e clawdbot_install_mode=development \
  -e clawdbot_repo_url=https://github.com/YOUR_USERNAME/clawdbot.git \
  -e clawdbot_repo_branch=feature-branch
```

#### Tailscale Auto-Connect

```bash
ansible-playbook playbook.yml --ask-become-pass \
  -e tailscale_authkey=tskey-auth-xxxxxxxxxxxxx
```

## License

MIT - see [LICENSE](LICENSE)

## Support

- Clawdbot: https://github.com/clawdbot/clawdbot
- This installer: https://github.com/ivozilkenat/moltbot-ansible/issues
