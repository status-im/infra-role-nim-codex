# Description

This role provisions a nim Codex installation.

# Introduction

The role will:

* Checkout a branch from the [nim-dagger](https://github.com/status-im/nim-dagger/) repo
* Build it using the [`build.sh`](./templates/build.sh.j2) Bash script
* Schedule regular builds using [Systemd timers](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
* Start a node by defining a [Systemd service](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

# Ports

The service exposes two ports by default:

* `9000` - LibP2P peering port. Must __ALWAYS__ be public.
* `9900` - Prometheus metrics port. Should not be public.

# Installation

Add to your `requirements.yml` file:
```yaml
- name: infra-role-nim-codex
  src: git+git@github.com:status-im/infra-role-nim-codex
  scm: git
```

# Configuration

The crucial settings are:
```yaml
# branch which should be built
nim_codex_repo_branch: 'stable'
# optional setting for debug mode
nim_codex_log_level: 'DEBUG'
```

# Management

## Service

Assuming the `main` branch was built you can manage the service with:
```sh
sudo systemctl start codex-main
sudo systemctl status codex-main
sudo systemctl stop codex-main
```
You can view logs under:
```sh
tail -f /data/codex-main/logs/service.log
```
All node data is located in `/data/codex-main/data`.

## Builds

A timer will be installed to build the image:
```sh
 > sudo systemctl list-units --type=service '*codex-*'
  UNIT                                LOAD   ACTIVE SUB     DESCRIPTION
  codex-prater-stable.service   loaded active running Codex (stable)
```
To rebuild the image:
```sh
 > sudo systemctl start build-codex-main
```
To check full build logs use:
```sh
journalctl -u build-codex-main.service
```

# Requirements

Due to being part of Status infra this role assumes availability of certain things:

* The `iptables-persistent` module
