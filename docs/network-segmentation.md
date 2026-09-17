# Network Segmentation

**Status: planned, not yet live.** Everything in this lab currently runs on the flat household network (`vmbr0`). This doc describes the target design and what's blocking the cutover.

## Target design

- **3070 SFF** takes over as the network edge, running OPNsense as firewall/router — this role was originally planned for the 7040 Micro, moved to the SFF so the 7040 could stay dedicated to lighter Proxmox workloads instead.
- **VLAN 10 (Trusted):** the existing household router, demoted to AP-only mode, plus household Wi-Fi/devices.
- **VLAN 20 (Lab/Gaming):** the Proxmox cluster (3070 Micro + 7040 Micro) and gaming devices.
- A managed switch with 802.1Q VLAN tagging sits between the edge and both VLANs.

See the [network topology diagram](../diagrams/network-topology.md) for the full layout.

## Why it's not live yet

**Physical cabling.** The modem and the existing household router/AP are colocated in one room; the rack is in another, connected today by exactly one long cable (currently router→PC). That cable gets repurposed as the new WAN leg (modem→SFF), which is straightforward. The harder part: the household router is staying at the modem-side location for Wi-Fi coverage but getting demoted to AP-only, which still needs a LAN/trunk connection back to the switch — and there's no second cable run between the two rooms yet. Options under consideration: run a second long cable, relocate the AP to the rack (if coverage from that room is still good enough for the house), or bridge it wirelessly (mesh/WDS) as a fallback.

**Hardware.** The SFF's second NIC (needed for a separate WAN + LAN-trunk port) hasn't been sourced yet — SFF-form-factor Dells usually have a PCIe slot, so a proper PCIe NIC card is being evaluated over a USB adapter, pending confirming the SFF's actual slot situation.

**ISP timing.** Currently on Spectrum; a possible switch to Frontier Fiber is being evaluated on price, not locked. The cutover will be planned around whichever ISP is actually in place when the network work happens, since a fiber ONT hand-off to OPNsense has known DHCP/WAN-lease quirks worth testing directly rather than assuming plug-and-play.

## What this demonstrates

Planning a segmented network around real physical constraints (existing cable runs, AP coverage, hardware slots) instead of a diagram that ignores them, and sequencing a migration around dependencies that aren't fully resolved yet rather than rushing a cutover.
