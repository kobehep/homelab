# Case Study: Recognizing a Recurring Failure Pattern on a Windows Client

## Situation

Domain-joining CLIENT01 (Windows 11) to `lab.local` was failing, with DNS resolution behaving inconsistently against the domain controller.

## Investigation

Running `nslookup` against the domain showed it silently returning results from Spectrum's public IPv6 DNS server instead of the domain controller — despite the client being configured to use the DC's IPv4 address for DNS. Windows prefers IPv6 over IPv4 by default whenever both are configured on an interface, so the client was quietly using a public IPv6 resolver instead of the internal DNS server it was actually pointed at.

This was recognizable almost immediately as the same class of problem as an issue hit the day before during [Proxmox cluster formation](proxmox-cluster-ipv6-masking.md): IPv6 auto-configuration silently overriding an intended IPv4 path.

## Root cause

IPv6 was enabled and auto-configured on the client NIC, and Windows' IPv6-preference behavior caused it to bypass the intended IPv4 DNS server for one it had no business using in an internal lab network.

## Corrective action

Disabled IPv6 entirely on the client's NIC — this lab network doesn't use IPv6 for anything internal, so there was no tradeoff in turning it off. Domain join then succeeded. (A secondary "primary DNS suffix" warning appeared during the join itself; confirmed cosmetic and non-fatal by logging in afterward as `administrator@lab.local`.)

## Lessons

- Recognizing a repeat of a known failure pattern turned what could have been another multi-step diagnosis into a five-minute fix.
- Decided to disable IPv6 by default on new lab VMs/nodes going forward rather than rediscovering this a third time — a small standing policy change based on two real incidents, not a hypothetical one.
