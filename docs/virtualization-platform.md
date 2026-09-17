# Virtualization Platform

## Platform summary

I run a 2-node Proxmox VE cluster: the 3070 Micro (32GB RAM, i5-9500T) and the 7040 Micro (12GB RAM, upgrading to 32GB, i5-6500T). Both nodes were joined into a single cluster so they can be managed from one dashboard, even though I run without automatic failover — see Quorum below.

## Quorum and HA — a deliberate tradeoff

A 2-node cluster only has 2 corosync votes total. If either node drops, the survivor loses quorum (it needs >50% of votes) and cluster-wide management freezes, even though any VM already running on the surviving node keeps running locally. The standard fix is a QDevice tie-breaker (commonly a Raspberry Pi running `corosync-qnetd`) to get back to an odd number of votes.

I chose to skip that for now and manage VM/LXC placement manually instead. For a solo lab, automatic failover wasn't worth the added complexity — but I documented the tradeoff rather than just not noticing it, and the QDevice fix is a known next step if that changes.

## Finding the real bottleneck: CPU, not RAM

My original capacity plan assumed RAM was the constraint on the 7040 Micro (it shipped with only 12GB against the 3070 Micro's 32GB). Checking the actual CPUs told a different story: both 3070s run an i5-9500T (6C/6T, no hyperthreading), while the 7040 Micro runs an older i5-6500T (4C/4T, no hyperthreading). Even after the RAM upgrade, the 7040 stays the weaker node — CPU headroom, not RAM, is the real long-term limit.

That changed my workload placement plan: CPU-hungry workloads (the AD lab, the Valheim LXC) stay on the 3070 Micro permanently, and the 7040 is reserved for light, low-CPU services once it's in use.

## Current resource allocation

On the 3070 Micro: AD DC (2 vCPU) + AD Client (2 vCPU) + Valheim LXC (4 vCPU) = 8 vCPU against 6 physical cores — a 1.33x oversubscription. I judged this acceptable because the DC and client sit idle almost all the time; real contention would only show up if Valheim spiked while the DC was also busy, which doesn't happen in a single-user lab.

The 7040 Micro is cluster-joined but not yet hosting workloads — it's waiting on its RAM upgrade before I put anything on it.

## What this demonstrates

Capacity planning based on measured hardware constraints instead of the first assumption, and making (and writing down) a deliberate availability tradeoff instead of defaulting to "add more infrastructure."
