# SystemBackupOptimize

Windows pre-backup readiness tool for Veeam Agent for Microsoft Windows. Read `docs/PROJECT-BRIEF.md` (full brief) and `docs/decisions/` before proposing anything.

## Current state

Planning only. Stages 1 and 2 of the brief are done. Next: Stage 3 (feature inventory, workflow, risk register), then Stage 4 (architecture), then Stage 5 (roadmap, tests, names, sources).

- Work one stage at a time. Stop at the end of each stage and wait for the owner's "go".
- **Do not write implementation code until Stage 5 is approved.** Short command examples used as evidence are fine.
- Record each accepted decision as a numbered file in `docs/decisions/`.

## Scope

- Windows 11 and Windows 10 22H2 client; Home, Pro, Enterprise (detect, never assume Pro features). Windows Server is out of scope.
- Veeam Agent standalone and VBR-managed. Free and Workstation editions have no script hook; the verdict there is advisory.
- Workgroup, on-prem AD, Entra ID joined, hybrid joined.
- Windows PowerShell 5.1 must work. PowerShell 7.x optional. No syntax or cmdlets that need 7.
- Reports in English and French.

## Design decisions already taken

- Two modes: **check** (read-only, right before backup, gives the verdict) and **maintenance** (conditional repairs and cleanup, hours ahead, reboot completed, fresh check afterwards).
- Default is assessment only. Repairs run only on a detected trigger condition; cleanup is never on by default.
- Statuses: Ready for backup / Ready with warnings / Not recommended for backup / Assessment incomplete. A failed or skipped critical check can never produce "Ready".
- Output contract: human-readable report, JSON report, and one documented process exit code per status.

## Hard rules (not negotiable)

- No registry cleaning. No "optimization" tweaks, despite the repo name.
- Never disable or weaken a security feature.
- Never delete a file whose purpose cannot be established. Personal files, user documents, application data and recovery partitions are off-limits.
- Never delete shadow copies or restore points.
- Prefer Windows-native, officially supported tools.
- Keep diagnostic evidence. Log every command with parameters, exit code, output and duration.
- Flag every operation that needs a reboot. Never reboot unattended.
- Verify results; never report success for a failed or inconclusive operation.

## Evidence rules

- Source priority: Microsoft Learn, then Veeam official docs, then vendor docs, then engineering publications, then inspectable open-source projects.
- Every non-trivial technical claim needs a direct link to a page actually opened. Otherwise label it **[UNVERIFIED]**.
- Certainty labels, exactly: **Verified**, **Strongly supported**, **Conditional** (name the conditions), **Uncertain** (say what would resolve it).
- Show disagreements between sources. Never invent references, version numbers or test results.

## Decision principle

Technical correctness > system integrity > backup reliability > evidence > maintainability > convenience, speed, feature count, aggressive cleanup or looks.

## Conventions (for when implementation starts)

- Challenge the owner's assumptions when evidence points elsewhere; do not simply agree.
- Owner prefers concise, copy-paste-ready scripts and fixes applied in batches.
- Pester for tests, PSScriptAnalyzer for linting, GitHub Actions for CI, SemVer and a changelog for releases. Details are fixed in Stage 4.
