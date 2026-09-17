# Service Platform

## What it is

A dedicated Valheim game server, run as an LXC container on the 3070 Micro and open to real external players — chosen as a "real service with real users" workload, not just an idle test VM.

## Build decisions

**LXC over VM.** Containers are much lighter on RAM/disk than a full VM for a workload like this, and Valheim doesn't need kernel-level isolation from the host.

**Rejected a low-trust install path.** I looked for an existing Proxmox community script to automate the build — checked both the `community-scripts/ProxmoxVE` helper-script project and the archived `tteck/Proxmox` repo, and neither had a Valheim script. The only third-party option I found was a single-maintainer, 1-star repo, which I judged too low-trust to curl-pipe-bash as root. I built it manually instead: a Debian LXC, SteamCMD installed by hand, running under a dedicated non-root `steam` user — the standard documented path, and one I could actually audit before running.

**Resilience via systemd, later superseded by a managed platform.** The initial build ran as a systemd service (`Restart=on-failure`) so it would survive console disconnects and come back after a crash or reboot. I later migrated it to CubeCoders AMP for web-based management (mods, backups, restarts) instead of doing everything by hand over SSH — after reviewing AMP's installer script first, since it wasn't a script I already trusted. The migration carried the real world save across rather than starting fresh, and I confirmed via the server logs that it loaded the existing world instead of silently generating a new one.

## Hardening

SSH access is key-only: the default root password was reset, my key was added to `authorized_keys`, and `PasswordAuthentication no` was set in `sshd_config` afterward.

## Incidents

Three separate bugs came out of the AMP migration and mod setup — a dashboard setting producing a malformed launch flag, a management UI shipping with a hardcoded `localhost` config, and a permissions system with two separate admin lists that both needed maintaining. Full writeups: [valheim-portal-modifier-flag.md](../case-studies/valheim-portal-modifier-flag.md), [amp-authserver-url-bug.md](../case-studies/amp-authserver-url-bug.md), and [server-devcommands-dual-permissions.md](../case-studies/server-devcommands-dual-permissions.md).

## Backups

Nightly Proxmox-level backup job (snapshot mode, 4:00 AM, keep-last 7) — see [backup-and-recovery.md](backup-and-recovery.md) for status.

## What this demonstrates

Evaluating third-party scripts for trust before running them as root, building a service manually when the "easy" path wasn't safe, migrating a stateful service without losing data, and hardening remote access as a matter of course rather than an afterthought.
