# Home Lab

I built and operate this three-node home lab to develop practical, hands-on experience in virtualization, Active Directory administration, network segmentation, and service operations.

**Status:** active, in progress. The virtualization platform, AD lab, and a live game server are all running; network segmentation is designed but not yet cut over — see [Roadmap](#roadmap).

## Architecture at a glance

```mermaid
flowchart TD
    Internet[Internet] --> Edge[OPNsense Firewall/Router<br/>3070 SFF]
    Edge --> Switch[Managed Switch<br/>802.1Q VLAN trunking]
    Switch --> AP[Household Router<br/>AP mode — VLAN 10]
    Switch --> PVE1[Proxmox Node 1 — 3070 Micro<br/>32GB RAM, i5-9500T]
    Switch --> PVE2[Proxmox Node 2 — 7040 Micro<br/>12→32GB RAM, i5-6500T]

    PVE1 --> DC01[DC01<br/>AD DS / DNS, lab.local]
    PVE1 --> CLIENT01[CLIENT01<br/>Windows 11, domain-joined]
    PVE1 --> Valheim[Valheim LXC<br/>CubeCoders AMP]
    PVE2 --> Future[Services pending —<br/>RAM upgrade first]

    classDef edge fill:#1f6feb,color:#ffffff,stroke:#0d419d;
    classDef platform fill:#238636,color:#ffffff,stroke:#146c2e;
    classDef guest fill:#8250df,color:#ffffff,stroke:#6639ba;
    classDef pending fill:#8c6500,color:#ffffff,stroke:#bf8700;
    class Edge,Switch,AP edge;
    class PVE1,PVE2 platform;
    class DC01,CLIENT01,Valheim guest;
    class Future pending;
```

This is the target topology — the OPNsense/VLAN cutover hasn't happened yet, so today everything still runs on the flat household network. Full explanation: [network-segmentation.md](docs/network-segmentation.md).

## What I built

- A 2-node Proxmox VE cluster with a documented, deliberate quorum/HA tradeoff (no QDevice, manual placement) rather than an unexamined default
- An Active Directory lab: a Windows Server 2022 domain controller and a domain-joined Windows 11 client, hitting and fixing a real DNS/IPv6 resolution bug along the way
- A production-managed Valheim dedicated game server (CubeCoders AMP), open to real external players, migrated from a manual build without losing world data
- A capacity plan revised after benchmarking actual hardware (CPU, not RAM, turned out to be the real constraint on the older node)
- A planned VLAN-segmented network behind an OPNsense firewall, scoped around real physical constraints (existing cable runs, AP placement, available NIC slots)

## Technical focus

| Area | Skills demonstrated |
|---|---|
| Virtualization | Proxmox VE clustering, quorum/HA tradeoffs, LXC vs. VM placement decisions, capacity planning against real CPU/RAM constraints |
| Systems administration | Windows Server 2022 AD DS/DNS, domain join, VirtIO drivers, UEFI/TPM guest requirements, SSH key-only hardening |
| Networking | VLAN segmentation design, 802.1Q trunking, DNS resolution troubleshooting, IPv6/IPv4 interaction issues |
| Operations | Service migration without data loss, backup job configuration, third-party script trust evaluation before running as root |
| Troubleshooting | Diagnosing from live process state and vendor source/config rather than assumptions, recognizing recurring failure patterns across systems |

## Portfolio guide

### Design and operations

| Document | What it demonstrates |
|---|---|
| [Architecture overview](docs/architecture-overview.md) | End-to-end design and priorities behind the lab |
| [Virtualization platform](docs/virtualization-platform.md) | Proxmox cluster design, quorum tradeoff, CPU-based capacity replanning |
| [Active Directory lab](docs/active-directory-lab.md) | DC/client build, AD DS/DNS roles, domain join troubleshooting |
| [Service platform](docs/service-platform.md) | Valheim/AMP build, script trust evaluation, stateful migration |
| [Network segmentation](docs/network-segmentation.md) | VLAN/OPNsense design and what's actually blocking the cutover |
| [Backup and recovery](docs/backup-and-recovery.md) | What's backed up, what's not yet restore-validated, and why that distinction matters |

### Case studies

| Case study | What it demonstrates |
|---|---|
| [Proxmox cluster IPv6 masking](case-studies/proxmox-cluster-ipv6-masking.md) | Diagnosing a cluster-join failure down to an IPv6/IPv4 config root cause |
| [AD domain-join IPv6 masking](case-studies/ad-domain-join-ipv6-masking.md) | Recognizing a recurring failure pattern across two different systems |
| [AMP AuthServerURL bug](case-studies/amp-authserver-url-bug.md) | Tracing a broken management UI to a bad default config value |
| [Valheim portal-modifier flag bug](case-studies/valheim-portal-modifier-flag.md) | Verifying a dashboard setting against the live process instead of trusting the UI |
| [Server Devcommands dual-permissions bug](case-studies/server-devcommands-dual-permissions.md) | Diagnosing two permission systems that needed to agree |

## Technology used

Proxmox VE · Windows Server 2022 (AD DS, DNS) · Windows 11 · OPNsense (planned) · CubeCoders AMP · SteamCMD · systemd · Debian LXC · VirtIO · Git/GitHub

## Roadmap

- [ ] Complete OPNsense/VLAN network cutover
- [ ] RAM upgrade (7040 Micro, 12GB → 32GB) and SSD storage installs across all three nodes
- [ ] PowerShell AD automation — scripted user provisioning (CSV → AD user/group/OU, with error handling and logging)
- [ ] Backup restore validation — actually restore a VM from the existing nightly backup and document the process
- [ ] Lightweight SIEM (Wazuh) — centralize and monitor AD/service logs for authentication anomalies
