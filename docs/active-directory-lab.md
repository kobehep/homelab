# Active Directory Lab

## What it is

A domain controller and a domain-joined client, built to get real practice with AD DS, DNS, and domain join — not just a tutorial walkthrough.

- **DC01** — Windows Server 2022 Standard (Desktop Experience), 2 vCPU, 4GB RAM, 64GB VirtIO SCSI disk, on the 3070 Micro. Promoted to domain controller for a new forest, domain `lab.local`. AD DS and DNS roles confirmed via Server Manager and by opening Active Directory Users and Computers cleanly.
- **CLIENT01** — Windows 11, 2 vCPU, 4GB RAM, 64GB VirtIO SCSI disk, also on the 3070 Micro. Built with the q35 machine type, OVMF (UEFI), and TPM v2.0 — all required for a Windows 11 install under Proxmox. Domain-joined to `lab.local`.

Both run static IPs, with the client pointed at the DC for DNS.

## Build notes

VirtIO drivers (network + disk) were loaded during setup from the official VirtIO Windows driver ISO rather than relying on emulated hardware — better performance, and it's how you'd actually run Windows guests under KVM/Proxmox in practice, not just how to get a VM to boot.

## The domain-join issue

Domain-joining CLIENT01 hit a networking problem that traces back to the same root cause as an issue in the [Proxmox cluster case study](../case-studies/proxmox-cluster-ipv6-masking.md): Windows prefers IPv6 when both IPv6 and IPv4 are configured, so `nslookup` was silently querying an external public DNS server over IPv6 instead of the domain controller. Full writeup: [ad-domain-join-ipv6-masking.md](../case-studies/ad-domain-join-ipv6-masking.md).

A secondary "primary DNS suffix" warning appeared during the join itself — confirmed cosmetic/non-fatal by successfully logging in afterward as `administrator@lab.local`, rather than assuming an error message meant the join had failed.

## What this demonstrates

AD DS/DNS role deployment, GPO-ready domain join, Windows Server virtualization requirements (UEFI/TPM for modern guest OSes), and diagnosing a DNS resolution failure down to a protocol-preference root cause rather than just restarting things until it worked.
