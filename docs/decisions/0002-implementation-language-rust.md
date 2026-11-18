# 0002: Implementation language is Rust

Date: 2026-11-18
Status: accepted (owner decision; Stage 4 still settles structure, crates and the open questions below)

## Decision

The tool is written in Rust and ships as one self-contained Windows executable. This replaces the PowerShell baseline in the brief (Windows PowerShell 5.1 required, Pester, PSScriptAnalyzer).

## Why

- **Owner goal:** the owner wants to learn Rust through this project.
- **No runtime on the target machine:** one `.exe`, so no PowerShell 5.1 vs 7, execution policy or module path differences. This meets the intent of the old "PowerShell 5.1 must work" rule (nothing to install) better than PowerShell itself.
- **The compiler can enforce the verdict rules:** statuses and check results become types, so code that turns a failed or skipped critical check into "Ready" can be made not to compile.
- **Short feedback loop for AI-assisted code:** strict compiler errors are precise and machine-actionable. Source of the argument: Underscore_ interview with Quentin Adam (Clever Cloud), https://youtu.be/AiytemqB_F0, transcript supplied by the owner. **Uncertain:** one practitioner's account, not measured data.

## Costs accepted

1. Most checks still rely on Windows tools (DISM, chkdsk, vssadmin, fsutil, reagentc, Veeam CLI), WMI/CIM and event logs. Rust must either run those tools and parse their output, or call the Windows APIs directly. Both take more code than PowerShell.
2. A technician cannot read an `.exe` the way they can read a `.ps1`. Transparency has to come from the report, the JSON output and the command log (already required by the hard rules), plus published source.
3. Code signing becomes close to mandatory for distribution. **[UNVERIFIED]** that unsigned executables are blocked more often than unsigned scripts by SmartScreen, Defender, AppLocker and WDAC; to be checked in Stage 4.
4. Two build targets: x64 and ARM64 Windows.

## Conditions (because the owner cannot yet review Rust)

The owner is learning Rust and cannot yet judge the code. The safety net therefore does not rely on owner code review:

1. **Read-only first.** The first releases contain check mode only. Maintenance mode (repairs, cleanup) is built after check mode is stable, and each repair requires: a written spec, tests including failure cases, a run on Hyper-V test VMs from a checkpoint, and a plain-language walkthrough of its code with the owner.
2. **The owner reviews what they can judge:** specs, test scenarios written as plain Given/When/Then, the reports and exit codes, and results on their own test VMs.
3. **Automated gates on every pull request:** `cargo fmt --check`, `cargo clippy` with warnings as errors, `cargo test`, dependency audit (for example `cargo audit` / `cargo deny`), Windows build for both targets.
4. **No `unsafe` outside one isolated module** for Windows API calls, enforced with `#![forbid(unsafe_code)]` everywhere else.
5. **Every system call goes through one interface** that tests can replace, so failure scenarios (failing VSS writer, full disk, stopped service, pending reboot, timeout, unexpected tool output) run in CI without touching Windows.
6. **Independent review:** a second AI review pass on each pull request, separate from the session that wrote the code.
7. **Teaching is part of the work:** each pull request explains in plain language what the code does and which Rust concepts it uses, so the owner's review ability grows with the project.

## Open for Stage 4

- Pure Rust, or a Rust core with small readable PowerShell 5.1 helper scripts for Windows calls (PowerShell 5.1 is built into every supported Windows version).
- Minimum supported Rust version, crates (Windows API bindings, WMI, JSON, localization), and how each is vetted.
- Code signing method and cost.
