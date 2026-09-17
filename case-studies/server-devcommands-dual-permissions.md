# Case Study: A Mod Kept Its Own Permissions List, Separate From the Base Game's

## Situation

Friends added to Valheim's admin list weren't getting the elevated command access a server mod (Server Devcommands) was supposed to grant them, despite being correctly listed as admins.

## Investigation

The base game reads `adminlist.txt` for admin status (Steam64 IDs, requires a server restart to take effect — it's only read at startup, not live). Server Devcommands, however, keeps its own separate `permissions.yaml`: it auto-creates a per-player entry the first time someone connects, but that entry isn't marked as an admin by default, regardless of what `adminlist.txt` says. Granting real devcommand access required an explicit `admin: yes` line added to that player's entry by hand.

## Root cause

Two independent permission systems — the base game's admin list and the mod's own permissions file — needed to agree, and only one of them was being maintained.

## Corrective action

Established a repeatable process: each friend's `permissions.yaml` entry appears automatically the first time they connect (no restart needed for that part, and it persists on disk after they disconnect); admin grants can then be batched later by editing `admin: yes` into each confirmed entry and restarting once, rather than needing everyone online at the same time. Verified each grant by reading the actual file on disk after the restart, rather than assuming a config edit had taken effect.

## Lessons

- A mod that extends a game's functionality doesn't necessarily extend its permission model too — check whether it's reading the same source of truth or keeping its own.
- Confirming state from the actual file on disk, not from an assumption that an edit + restart worked, caught this before it became a repeated support request from friends.
