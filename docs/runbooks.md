# Runbooks — Real Problems Hit

Real incidents from building the [homelab](../README.md), diagnosed and fixed. Each of these came from reading logs, checking live process state, or reading vendor source/config directly — not guessing.

## 1. IPv6 silently masked a missing IPv4 config during cluster formation
Proxmox's Join Cluster dialog only lists statically-configured addresses — a DHCP lease doesn't qualify as a stable corosync link. One node had no IPv4 configured at all (IPv6-only), and `/etc/hosts` referenced a stale IPv6 address that SLAAC had already reassigned. **Fix:** set static IPv4 on all cluster nodes before touching cluster networking, and treat IPv6 SLAAC addresses as unstable by default for anything infrastructure-critical.

## 2. Same class of bug, second occurrence, on a Windows client
While domain-joining a Windows 11 client, `nslookup` was silently querying Spectrum's public IPv6 DNS server instead of the domain controller — Windows prefers IPv6 when both are configured. **Fix:** disabled IPv6 on the client NIC. Recognizing the pattern from incident #1 turned a second confusing bug into a five-minute fix.

## 3. A dashboard setting silently produced a broken launch flag
Enabling a game server's "portal restriction" setting through the management UI produced a malformed launch flag when checked against the live process command line. **Fix:** read the platform's own template/config source directly to find the actual expected value, rather than trusting the dashboard label — confirmed the fix against the running process, not just the saved setting.

## 4. A management UI's default config pointed at the wrong host
A game-server management platform shipped with its auth callback hardcoded to `localhost`, which resolves to the browser's machine, not the server — breaking the management dashboard flow. Traced it to the platform's own config key and corrected it to the LXC's real LAN IP via the platform's CLI reconfiguration tool.

## 5. A permissions system had two separate admin lists that both needed maintenance
A game server mod checked its own permissions file (auto-created per player on first connect, but not marked as admin by default) separately from the base game's admin list. Granting real admin access required updating both, and confirming via the actual file on disk — not assuming a UI action had taken effect — before restarting the service.

## Lessons Learned

The recurring theme across this build has been **verifying real state instead of trusting a UI, a default, or an assumption** — checking the live process command line, reading a vendor's own template file, re-reading a config after a restart to confirm a change actually persisted. Every incident above got solved faster the second time a similar pattern showed up, which is the main argument for writing them down at all.
