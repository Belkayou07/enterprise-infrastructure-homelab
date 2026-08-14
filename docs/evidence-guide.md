# Evidence Capture Guide

This project uses screenshots as supporting evidence, not as a replacement for documentation. Evidence should prove that a real configuration, verification, or troubleshooting step was performed.

## Rule

Each chapter must contain explicit **EVIDENCE CHECKPOINTS**. When a checkpoint is reached, capture the requested screenshot before moving on when practical.

Do not screenshot every command. Capture only states that materially prove the work.

## What Makes a Good Screenshot

A useful screenshot should:

- show the relevant command and its result, or the relevant management console state;
- include enough context to understand what is being proven;
- avoid passwords, product keys, tokens, personal email addresses, unrelated private IPs, and other secrets;
- avoid excessive desktop clutter when possible;
- be named consistently.

## Naming Convention

```text
chapter-step-short-description.png
```

Examples:

```text
00-03-smt-24-logical-processors.png
01-01-hyperv-verification.png
01-03-virtual-switches.png
04-02-dc01-domain-controller.png
```

## Evidence Levels

```text
DESIGN EVIDENCE
Architecture diagrams, schemas, addressing plans, engineering decisions

IMPLEMENTATION EVIDENCE
Configuration screens, commands that create resources, deployed systems

VERIFICATION EVIDENCE
Commands or consoles proving the intended state actually exists

TROUBLESHOOTING EVIDENCE
Before/after states and the key observation that identified a real fault
```

## Chapter 0 — Retroactive Evidence Recovery

Useful evidence that can still be captured now:

1. `00-01-host-cpu-memory.png`
   - Task Manager -> Performance -> CPU
   - show Ryzen 9 7900, 12 cores, 24 logical processors
   - RAM may be visible in a separate screenshot if useful

2. `00-03-smt-24-logical-processors.png`
   - PowerShell output showing:
   - `NumberOfCores = 12`
   - `NumberOfLogicalProcessors = 24`
   - `ThreadCount = 24`

3. Architecture and IP-plan evidence is already represented as version-controlled documentation/schemas and does not require screenshots simply for appearance.

The earlier Ryzen Master `SMT = OFF` state was observed during troubleshooting. If the original screenshot is available, it is useful as before-state evidence, but it should not be recreated artificially.

## Chapter 1 — Current Evidence Checkpoints

### Checkpoint 1.1 — Hyper-V installation verification

Capture:

```text
01-01-hyperv-verification.png
```

The PowerShell window should show, preferably in one screenshot:

- `Microsoft-Hyper-V-All = Enabled`
- `vmms = Running / Automatic`
- `Get-VM` from module `Hyper-V`
- `Default Switch = Internal`

### Checkpoint 1.2 — Storage convention

Capture after the storage paths have been deliberately configured and verified, not before.

### Checkpoint 1.3 — Virtual-switch inventory

Capture after `vSW-USERS`, `vSW-SERVERS`, and `vSW-MGMT` have been created and verified.

### Checkpoint 1.4 — Test VM

Capture the Hyper-V Manager or PowerShell inventory proving the test VM exists and is running, plus one guest-side verification if it adds value.

## Portfolio Principle

A reviewer should be able to follow this chain:

```text
DESIGN
  |
  v
IMPLEMENT
  |
  v
VERIFY
  |
  v
EVIDENCE
  |
  v
EXPLAIN
```

Evidence should be concise, credible, and tied to a documented result.