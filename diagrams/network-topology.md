# Network Topology

This is the lab's target architecture. Flat (no nested boxes) so it renders cleanly, with color grouping instead of nested subgraphs: blue for edge/network infrastructure, green for the Proxmox platform and its guests, gold for what's still pending.

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

## Current state vs. target

This diagram is the **target** topology, not what's live today. Right now the lab still runs on the flat household network (`vmbr0`) — the OPNsense/VLAN cutover hasn't happened yet, pending a second NIC for the edge box and more network cabling. The Proxmox cluster, AD lab, and Valheim server are all live regardless; only the edge/segmentation layer is still pending. See [network-segmentation.md](../docs/network-segmentation.md) for the cutover plan.
