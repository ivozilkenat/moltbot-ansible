# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial release
- Tailscale VPN installation and configuration
- UFW firewall with Docker isolation (DOCKER-USER chain)
- Docker CE + Compose V2 installation
- Node.js 22.x + pnpm installation
- Clawdbot container setup with non-root user
- Systemd service for auto-start
- Localhost-only port binding (127.0.0.1:3000)
- One-command installation script
- Comprehensive documentation (security, architecture, troubleshooting)
- Ansible collections requirements (community.docker, community.general)

### Security
- UFW firewall enabled by default (only SSH + Tailscale ports open)
- DOCKER-USER iptables chain prevents containers from exposing ports externally
- Container isolation: even `docker run -p 80:80` won't expose port 80
- Non-root container user
- Dynamic network interface detection (not hardcoded eth0)

## [0.1.0] - YYYY-MM-DD

Initial release placeholder.

[Unreleased]: https://github.com/pasogott/clawdbot-ansible/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/pasogott/clawdbot-ansible/releases/tag/v0.1.0
