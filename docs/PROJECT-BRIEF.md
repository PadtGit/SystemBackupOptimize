# Project: Universal Windows Pre-Backup Health, Repair & Maintenance Tool

You are acting as a senior Windows systems engineer, Rust developer and teacher, software architect, security reviewer and technical researcher. Challenge my assumptions whenever evidence points to a better approach. Do not simply agree with my design.

---

## 0. Context (confirmed 2026-11-14)

| Item | Value |
|---|---|
| Author / GitHub owner | PadtGit |
| Relationship to existing repo | Separate new repo (SystemBackupOptimize). PreFlightCheck (https://github.com/PadtGit/PreFlightCheck) is my existing project and serves only as an example of the idea. |
| Backup product | Veeam Agent for Microsoft Windows, both standalone and managed by Veeam Backup & Replication. Version: current release; note differences between recent major versions. |
| Run frequency | ~3 times per week, immediately before scheduled backups |
| Execution mode | Both: interactive (a technician watches) and unattended (scheduled task / Veeam pre-job script / RMM) |
| Target OS | Windows 11 and Windows 10 22H2 (client). Windows Server: out of scope |
| Editions | Home, Pro, Enterprise (detect; don't assume Pro features) |
| Environments | Workgroup, on-prem AD, Entra ID joined, hybrid joined |
| Implementation language | Rust, one self-contained `.exe` for x64 and ARM64 (decision 0002, 2026-11-18). Replaces the earlier PowerShell 5.1 baseline. |
| Report language | English and French |
| License | Not decided; recommend one in Stage 5 (MIT / GPL-3.0 / Apache-2.0 / other) |

---

## 1. Objective

Build an open-source, manufacturer-agnostic Windows tool that runs right before a Veeam Agent backup. It should:

1. find out whether the system is in a state that is fit to back up,
2. fix only the problems it actually detects, and only when the fix is safe and justified,
3. clean up only what is provably safe to remove,
4. give a clear, honest readiness verdict.

A healthy Windows system is not proof that a backup will succeed or can be restored. The tool improves the odds and reports the risks. It never promises a good backup.

---

## 2. Working method: staged, with stop gates

The deliverables are too large for one response. Work in stages. At the end of each stage, stop, summarize, and wait for my "go" before starting the next.

| Stage | Output | Stop gate |
|---|---|---|
| 1 | Clarifying questions (section 0) | My answers |
| 2 | Research and evidence matrix (section 4) | My review |
| 3 | Feature inventory, workflow, risk assessment (sections 5–7) | My review |
| 4 | Architecture and technology choice (section 8) | My review |
| 5 | Phased roadmap, testing strategy, open questions, project names (sections 9–11) | My approval |

Do not write implementation code in any stage. Short command examples used as evidence (for example, an exact DISM syntax) are fine.

---

## 3. Evidence rules

### Source priority
1. Microsoft Learn / official Windows and PowerShell documentation
2. Veeam official documentation (Help Center, KB articles, best-practice guides)
3. Hardware, storage and security vendor documentation
4. Recognized engineering research and publications
5. Established open-source projects with code you can inspect

### Citation rules
- Every non-trivial technical claim needs a direct link to the page that supports it.
- Only cite pages you actually opened in this session. If you're relying on prior knowledge you couldn't verify, label it **[UNVERIFIED]**.
- For each source, state in one line what it actually establishes. Don't just name the source.
- Never invent references, test results, version numbers or guarantees.
- When sources disagree, show the disagreement. Don't pick one silently.

### Certainty labels (use exactly these)
- **Verified:** authoritative documentation directly supports it, or it can be reproduced.
- **Strongly supported:** several credible sources agree, but some uncertainty remains.
- **Conditional:** correct only for certain Windows versions, hardware or situations. Name the conditions.
- **Uncertain:** not enough evidence. Say what test or source would resolve it, and recommend implement / modify / exclude.

Aim for the highest certainty you can actually reach. Never claim more certainty than the evidence supports.

---

## 4. Research scope (Stage 2)

For each operation below, fill in one row of an **evidence matrix** with these columns:

Operation | What it does | When it is justified (trigger condition) | Safe frequency (given ~3 runs/week) | Risk / side effects | Reboot needed? | Disk I/O cost | How to verify success | Exit codes / output to capture | Certainty label | Sources

### A. Assessment (read-only)
Windows version/build/edition; hardware inventory; volumes, free space, filesystem type, BitLocker state; storage health (Get-PhysicalDisk, Storage Reliability Counters, SMART where available, NVMe vs SATA vs HDD differences); filesystem dirty bit; component store state; pending reboot (every documented source); Windows Update / servicing activity in progress; critical Event Log entries (define WHICH event IDs/providers matter for backup readiness and why); relevant services; Defender / security status; time sync (if relevant to backups).

### B. Repairs (conditional)
DISM /ScanHealth, /CheckHealth and /RestoreHealth; SFC; CHKDSK (/scan, /spotfix, /f, /r: online vs offline behavior); service recovery. For each, answer:
- What detected condition justifies running it?
- What does it cost (time, I/O, SSD wear, reboot)?
- Is running it right before a backup counterproductive (heavy I/O, a reboot needed mid-window, changes made while VSS is about to snapshot)?

### C. Cleanup
Temp folders, Windows Update cache (SoftwareDistribution), component store cleanup (StartComponentCleanup with and without /ResetBase: note that /ResetBase is irreversible), Delivery Optimization cache, WER reports, logs, Recycle Bin, Storage Sense / cleanmgr equivalents.
- Sort each item into: safe to automate / needs approval / never automate.
- Check specifically whether any cleanup could hurt VSS or Veeam (for example, deleting shadow copies or restore points, or clearing caches Veeam uses).
- Personal files, user documents, application data and recovery partitions are off-limits. No exceptions.

### D. Veeam-specific readiness
Use Veeam's official documentation to establish:
- Which checks Veeam itself recommends or requires before an Agent backup (VSS writer state, VSS storage/space, CBT driver, Veeam services, target reachability, free space for snapshots).
- Whether Veeam Agent offers a supported pre-job script hook, plus its timeout, required exit codes, and how a non-zero exit affects the job.
- What can make an application-consistent backup fail, and which of those the tool can actually detect.

### E. Things I may be wrong about
Look for evidence against these assumptions and report what you find:
1. That running repair tools regularly improves backup reliability.
2. That cleanup before a backup is useful, rather than just shrinking the backup.
3. That repairs should run in the same session, right before the backup.
4. That my 8-step workflow (Review → Assessment → Repair Preview → Repair → Cleanup Preview → Cleanup → Final Review → Readiness) is the right order.

---

## 5. Feature inventory (Stage 3)

For each feature, give: purpose, category (read-only / safe-auto / conditional / approval-required / never-automate), trigger condition, evidence row reference, Windows-version/hardware conditions, and the priority you recommend (MVP / later / excluded).

For major decisions only (the operations in 4B, 4C and 4D, plus the architecture), add:
- **Primary recommendation:** why, how it works, pros/cons, risks, compatibility, how to verify it worked.
- **Alternatives (0–3):** how each works, when it would be better, why it isn't the primary choice. If fewer real alternatives exist, say so. Don't invent filler.

---

## 6. Execution workflow (Stage 3)

Propose the workflow you recommend, and justify any changes from my 8-step draft. It must:
- keep assessment separate from changes; default mode = assessment only,
- preview every change before running it,
- in interactive mode, ask for confirmation before risky operations,
- in unattended mode, define exactly which operations are allowed without a human, and skip and report everything else,
- stop when a critical prerequisite fails (define which ones are critical),
- support safe cancellation (define what "safe" means for each operation, especially one already running, like DISM or CHKDSK),
- log every command, its parameters, exit code, stdout/stderr and duration,
- verify results instead of assuming success,
- never report success for a failed or inconclusive operation,
- finish within a defined time budget that fits before the backup window (propose a budget).

---

## 7. Risk and safety assessment (Stage 3)

A risk register: risk, likelihood, impact, mitigation, residual risk. It must cover at least these risks:
- data loss
- an unbootable system
- a failed or inconsistent backup caused by the tool
- SSD wear from repeated runs
- a reboot during the backup window
- security features weakened
- running unattended without consent
- running on unsupported hardware or OS
- a false "Ready" verdict

Hard rules (not up for negotiation):
- no registry cleaning
- no "optimization" tweaks
- never disable security features
- never delete a file whose purpose can't be established
- prefer Windows-native, officially supported tools
- keep diagnostic evidence
- flag every operation that needs a reboot

---

## 8. Architecture (Stage 4)

The language is fixed: Rust (decision 0002). Compare pure Rust against a Rust core with small PowerShell 5.1 helper scripts, and cover:
- language/framework and the minimum runtime
- project/module structure and responsibilities
- configuration (a policy file that controls what is allowed per environment)
- admin/elevation handling
- logging
- error handling
- **output contract:** a human-readable report (HTML or Markdown) plus a machine-readable JSON report, plus **documented process exit codes**, one per readiness status, so Veeam, Task Scheduler or an RMM can act on the result
- localization (EN/FR)
- testing (`cargo test`, system calls behind a replaceable interface, failure-scenario simulation, test VMs across Win10/Win11 and editions, Hyper-V checkpoints for destructive tests)
- rustfmt and Clippy rules, dependency audit
- GitHub Actions CI
- code signing
- SemVer releases and a changelog

### Readiness statuses
- **Ready for backup**
- **Ready with warnings**
- **Not recommended for backup**
- **Assessment incomplete**

Define exactly which check results lead to each status. A failed or skipped critical check can never produce "Ready".

### Report contents
Date/time; computer ID; OS version/build/edition; tool version; initial findings; actions recommended; actions executed (command, parameters, exit code, duration); before/after comparison; warnings; errors; unresolved issues; final status with a one-sentence plain-language explanation that a non-technical user can understand.

---

## 9. Roadmap and testing (Stage 5)

Split implementation into phases. For each phase give: objectives, features, technical requirements, dependencies, risks, test requirements, and measurable completion criteria. A phase is done only when its criteria are met and I've reviewed it.

---

## 10. Model and effort recommendation (Stage 5)

Available models: Claude Opus 5.5, Claude Sonnet 5.5, Claude Haiku 5.5 (and Claude Fable 5.2 where available). Recommend a model and reasoning effort for: research, architecture, PowerShell/Windows development, security/safety review, implementation, code review, documentation, and testing/troubleshooting. Don't default to the most powerful model. Justify each choice by how hard and how risky the task is.

---

## 11. Remaining deliverables (Stage 5)

1. A consolidated list of every source used, grouped by priority tier.
2. Unresolved questions and uncertainties, each with what would resolve it.

(The project name is settled: SystemBackupOptimize.)

---

## 12. Decision principle

When in doubt, choose: technical correctness > system integrity > backup reliability > evidence > maintainability > convenience, speed, number of features, aggressive cleanup or looks.
