# 0001: Stage 1 answers and Stage 2 conclusions

Date: 2026-11-14
Status: accepted (Stage 2 conclusions pending owner review of the full report)

## Stage 1 answers (from the owner)

| Question | Answer |
|---|---|
| Veeam Agent deployment | Both standalone and managed by Veeam Backup & Replication |
| Execution mode | Both interactive and unattended |
| Windows Server | Out of scope |
| License | To be recommended in Stage 5 |
| Relationship to PreFlightCheck | Separate new project. PreFlightCheck is an example of the idea, not the codebase to continue. |
| Repository name | SystemBackupOptimize (chosen by the owner) |

## Stage 2 conclusions

1. **Two modes instead of one workflow.** A fast, read-only check runs before the backup and gives the readiness verdict. Repairs and cleanup move to a separate maintenance mode that runs hours ahead, with any reboot finished and a fresh check before the backup window.
2. **New repository.** PreFlightCheck's scope does not fit: it is built for one laptop model (Dell G5 5590), Windows 11 only, needs PowerShell 7 and includes WinGet updates. Its safety code is worth porting: preview before cleanup, age-gated temp deletion, tests and CI.
3. **Owner assumptions.** The evidence contradicts three of the four: running repairs routinely, running them right before the backup, and cleaning up by default. The fourth, the 8-step order, is partly right but needs a reboot step and a fresh check after any change.
4. **Veeam integration.** The Free and Workstation editions have no script hook, so on those the tool runs as a scheduled task that cannot stop the backup. The pre-freeze hook only exists in the Server edition and in jobs managed by Veeam Backup & Replication.

## Consequences

- Default mode is assessment only. Nothing is repaired or deleted unless maintenance mode is requested.
- On Free and Workstation editions the verdict is advisory: the report and exit code must say that the backup was not blocked.
- The name contains "Optimize", but the project's hard rules forbid optimization tweaks. The README states this so the name does not set the wrong expectation.

## Open

- Sources and certainty labels for each conclusion live in the Stage 2 report, which is not yet in `docs/research/`.
