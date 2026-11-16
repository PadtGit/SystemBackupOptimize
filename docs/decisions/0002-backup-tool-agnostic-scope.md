# 0002: Scope widened from Veeam Agent to any backup tool

Date: 2026-11-16
Status: accepted (owner request)

## Decision

The tool checks whether a Windows system is fit to back up before any backup tool runs, not only Veeam Agent for Microsoft Windows. Veeam Agent stays the first target.

## Consequences

- The readiness checks that concern Windows itself (disk, file system, VSS, component store, pending reboot) apply to any backup tool. Checks tied to one product (services, drivers, script hook, exit code handling) become per-product parts.
- Stage 2 research covered Veeam only. Nothing is established yet for other backup tools: which ones are in scope, whether they offer a pre-job script hook, and how they react to an exit code. Until that is researched, claims about other tools are **[UNVERIFIED]** and the verdict for them is advisory.
- The README states the wider scope and says that Veeam Agent is the only product researched so far.

## Open

- `docs/PROJECT-BRIEF.md` and `CLAUDE.md` still describe the project as a Veeam Agent tool. They need the same change.
- Which other backup tools to name and research, if any, before Stage 3.
