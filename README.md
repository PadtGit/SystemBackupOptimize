# SystemBackupOptimize

Open-source, manufacturer-agnostic Windows tool that checks whether a system is fit to back up before any backup tool (Veeam Agent or another product) runs its job, and gives an honest readiness verdict.

> **Status: planning.** No implementation code yet. Stage 2 (research) is done; Stage 3 (feature inventory, workflow, risk register) is next.

A healthy Windows system is not proof that a backup will succeed or can be restored. This tool improves the odds and reports the risks. It never promises a good backup.

Despite the name, this tool does no registry cleaning and applies no "optimization" tweaks.

## Scope

| Item | Value |
|---|---|
| Target OS | Windows 11 and Windows 10 22H2 (Home, Pro, Enterprise). Windows Server is out of scope. |
| Backup product | Any Windows backup tool. Veeam Agent for Microsoft Windows (standalone and managed by Veeam Backup & Replication) is the first target and the only one researched so far. |
| Execution | Interactive (technician) and unattended (scheduled task, the backup tool's pre-job script hook where it has one, RMM) |
| Environments | Workgroup, on-prem AD, Entra ID joined, hybrid joined |
| PowerShell | Windows PowerShell 5.1 required; PowerShell 7.x optional |
| Report languages | English and French |
| License | Not decided yet |

## Design in one paragraph

Two modes. **Check** is fast and read-only; it runs right before the backup and produces the readiness verdict. **Maintenance** holds the conditional repairs and cleanup; it runs hours ahead of the backup window, finishes any reboot, and is followed by a fresh check. See [docs/decisions](docs/decisions) for why.

## Readiness statuses

- Ready for backup
- Ready with warnings
- Not recommended for backup
- Assessment incomplete

A failed or skipped critical check can never produce "Ready".

## Documents

- [docs/PROJECT-BRIEF.md](docs/PROJECT-BRIEF.md): the full project brief and staged working method
- [docs/decisions/0001-stage-1-and-2-decisions.md](docs/decisions/0001-stage-1-and-2-decisions.md): decisions taken so far
- [docs/decisions/0002-backup-tool-agnostic-scope.md](docs/decisions/0002-backup-tool-agnostic-scope.md): scope widened from Veeam Agent to any backup tool
- [docs/research/](docs/research): Stage 2 research report and evidence matrix
- [CLAUDE.md](CLAUDE.md): working rules for Claude Code in this repo
