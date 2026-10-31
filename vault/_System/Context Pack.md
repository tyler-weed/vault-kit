---
type: system
---

# Context Pack

Read before any task that depends on a connector or a scheduled job.

## Connectors

| Connector | Used for | State | Notes |
|---|---|---|---|
| <!-- e.g. Email --> | | | |

## Scheduled tasks

Every job writes a heartbeat on every run (to `_System/Sweeps/` for sweeps, `_System/Brief/` for the
brief, `Backup Status.md` for the backup) so the dashboard can show silence.

| Job | When | Runs where | Writes | Heartbeat |
|---|---|---|---|---|
| <!-- e.g. Morning brief --> | | | | |

## Session read order

1. `CLAUDE.md`
2. `_System/Vault Map.md`
3. The project note for the work at hand
4. `_System/Agent Memory/MEMORY.md` if the task touches a repo, build, connector or git

## Standing cautions

<!-- Things that have bitten you before. Date each line. -->
