# Architecture Overview

## Why this lab exists

I built this to get hands-on with virtualization, Active Directory, and network segmentation beyond what's possible in a purely cloud/tutorial setting — real hardware, real failures, real fixes. See the [network topology](../diagrams/network-topology.md) for the target layout.

## What it's built on

Three repurposed Dell OptiPlex business desktops, picked specifically because they're cheap, quiet, and rack-friendly compared to full towers:

- **3070 SFF** — takes the network edge role (OPNsense firewall/router)
- **3070 Micro** — Proxmox node 1
- **7040 Micro** — Proxmox node 2

## Design priorities, in order

1. **Get real workloads running first, harden the network second.** The Proxmox cluster, AD lab, and a live game server all shipped before the VLAN/OPNsense segmentation did — I didn't want the network-perfection project to block getting hands-on with virtualization and AD.
2. **Match workload placement to real hardware constraints**, not assumptions. See [virtualization-platform.md](virtualization-platform.md) for how a RAM-based plan got revised once CPU turned out to be the real limit on the older node.
3. **Document decisions and incidents as they happen**, not after the fact — see the [case studies](../README.md#case-studies) for the specific problems this surfaced.

## Components

| Component | Role | Status |
|---|---|---|
| [Virtualization platform](virtualization-platform.md) | 2-node Proxmox VE cluster | Live |
| [Active Directory lab](active-directory-lab.md) | Windows Server 2022 DC + domain-joined client | Live |
| [Service platform](service-platform.md) | Valheim dedicated server, managed via AMP | Live |
| [Network segmentation](network-segmentation.md) | VLAN-separated lab network behind OPNsense | Planned |
| [Backup and recovery](backup-and-recovery.md) | Nightly Proxmox backups | Configured, restore-validation pending |
