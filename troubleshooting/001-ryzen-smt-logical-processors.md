# Troubleshooting 001 — Ryzen 9 7900 Exposed Only 12 Logical Processors

## Context

During the Chapter 0 host audit for my BelkaCorp homelab, I verified the physical CPU topology before enabling the full virtualization platform. The host uses an AMD Ryzen 9 7900, which has 12 physical cores and should expose 24 logical processors when SMT is enabled.

## Symptom

Windows reported:

```text
NumberOfCores               12
NumberOfLogicalProcessors   12
ThreadCount                 24
```

The physical core count was correct, but Windows exposed only 12 logical processors instead of the expected 24.

## Investigation

I checked the configuration chain in sequence instead of assuming a hardware fault:

1. I confirmed the installed CPU model.
2. I confirmed firmware virtualization was enabled.
3. I confirmed Windows was not boot-limited with a `numproc` setting.
4. I checked the MSI BIOS and found `SMT Control = Auto`, not Disabled.
5. I inventoried installed CPU and platform utilities.
6. I found AMD Ryzen Master installed.
7. I opened Ryzen Master and observed `Simultaneous Multithreading = OFF` while `Legacy Compatibility Mode = OFF`.

## Root Cause

SMT had been disabled through AMD Ryzen Master. As a result, Windows exposed one logical processor per physical core even though the CPU supports two hardware threads per core.

## Resolution

I changed `Simultaneous Multithreading` from `OFF` to `ON` in Ryzen Master, applied the change, and restarted Windows.

## Final Verification

After the restart, I verified the CPU topology again with PowerShell:

```text
NumberOfCores               12
NumberOfLogicalProcessors   24
ThreadCount                 24
```

This restored the expected 12-core / 24-logical-processor topology before I began allocating resources to Hyper-V virtual machines.

## Evidence

The actual evidence is committed in the repository:

- `screenshots/chapter-00/00-01-host-cpu.png`
- `screenshots/chapter-00/00-03-smt-24-logical-processors.png`

The first screenshot records the host CPU and virtualization state during the audit. The second records the corrected 12-core / 24-logical-processor PowerShell result after remediation.

## Lesson Learned

When observed CPU topology does not match the hardware capability, I should verify the full configuration chain rather than assuming a hardware problem:

```text
Hardware capability
        |
        v
BIOS / UEFI configuration
        |
        v
Windows boot configuration
        |
        v
Vendor tuning utilities
        |
        v
Final OS-visible topology
```

This incident also showed me why host validation should happen before VM resource planning: virtualization capacity decisions need to be based on the resources Windows actually exposes.