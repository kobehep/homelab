# Case Study: IPv6 Masking a Missing IPv4 Config During Cluster Formation

## Situation

Forming the 2-node Proxmox cluster failed on one node during the join step, with no obvious single error pointing at a root cause.

## Investigation

Proxmox's Join Cluster dialog only lists statically-configured addresses in its network dropdown — a DHCP lease doesn't count as a stable enough address for a corosync link. Checking that node's actual network config showed it had no IPv4 configured at all: its `vmbr0` bridge had a static IPv6 block only, with IPv4 never configured in the first place, not merely failing to get a DHCP lease. On top of that, `/etc/hosts` still mapped the node's own hostname to a stale static IPv6 address that SLAAC had already reassigned elsewhere — and Proxmox reads that file to resolve "this node's own address" during cluster operations.

A separate, related mistake surfaced along the way: the first node's own cluster had been created with an IPv6 link by default, which had to be torn down (`rm /etc/pve/corosync.conf` and `/etc/corosync/*`, restart `pve-cluster`) and recreated with IPv4 explicitly chosen for Link 0.

## Root cause

IPv6 SLAAC auto-configuration was masking the fact that IPv4 was either missing or unstable on both ends of the cluster link — the interfaces looked "configured" because they had an address, just not the one the tooling actually needed.

## Corrective action

Added an explicit `inet dhcp` stanza to the affected node's `vmbr0` config alongside its existing `inet6 static` block, then converted it to a static IPv4 address on the lab subnet once DHCP confirmed connectivity — since the Join Cluster dialog specifically needs a static entry to offer it as a link option. Recreated the cluster with IPv4 selected explicitly for Link 0. Confirmed via `pvecm status`: 2 nodes, 2/2 votes, quorate, both nodes on IPv4.

## Lessons

- Always set static IPv4 on Proxmox nodes before touching cluster networking — a DHCP lease won't show up as a valid corosync link option, whatever else looks fine.
- IPv6 SLAAC addresses can silently change out from under a config that assumes they're stable, breaking anything (like `/etc/hosts`) that captured one as a fixed reference.
- "The interface has an address" and "the interface has the address this specific feature needs" are different claims — worth checking which one you actually verified.

This exact failure pattern recurred a day later in a different context — see [AD domain-join IPv6 masking](ad-domain-join-ipv6-masking.md).
