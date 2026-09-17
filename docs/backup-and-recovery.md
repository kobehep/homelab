# Backup and Recovery

## What's configured

A nightly Proxmox-level backup job (Datacenter → Backup): 4:00 AM, snapshot mode, keep-last 7, targeting local storage. Confirmed there's plenty of headroom for it on the current single stock drive before relying on it.

## What's not done yet — and why that matters

The backup job has never been used to actually restore a VM. It runs, and the storage math works out, but neither of those things proves a restore would actually succeed — a backup job with a green checkmark and a backup that can actually bring a system back are not the same claim, and I haven't tested the second one yet. This is a deliberately tracked gap, not an oversight I'm unaware of: it's next on the roadmap specifically because "the job runs" is the easy 90% and "I restored a VM and watched it come back clean" is the part that actually proves anything.

## Plan

Restore one VM (likely DC01, since it's the lowest-risk to briefly duplicate) from an existing nightly backup into a test location, confirm it boots and the AD services come up clean, and document the actual restore steps and timing — not just that the backup job's log says "OK."

## What this demonstrates

Treating "backup completed" and "backup verified" as two different claims, and prioritizing proving recoverability over just accumulating more backup jobs.
