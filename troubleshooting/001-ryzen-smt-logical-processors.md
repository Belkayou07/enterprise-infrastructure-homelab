# Incident 001 — Ryzen 9 7900 Exposed Only 12 Logical Processors

## Symptom

During the Chapter 0 host audit, Windows reported an AMD Ryzen 9 7900 with:

```text
NumberOfCores               12
NumberOfLogicalProcessors   12
ThreadCount                 24
```

The expected logical processor count was 24.

## Investigation

The following checks were performed in sequence:

1. Confirmed the installed CPU model.
2. Confirmed firmware virtualization was enabled.
3. Confirmed Windows was not boot-limited with a `numproc` setting.
4. Checked the MSI BIOS and found `SMT Control = Auto`, not Disabled.
5. Inventoried installed CPU/platform utilities.
6. Found AMD Ryzen Master installed.
7. Opened Ryzen Master and observed `Simultaneous Multithreading = OFF` while `Legacy Compatibility Mode = OFF`.

## Root Cause

SMT had been disabled through AMD Ryzen Master, causing Windows to expose one logical processor per physical core.

## Resolution

`Simultaneous Multithreading` was changed from `OFF` to `ON` in Ryzen Master and applied. The system was restarted.

## Verification

After restart, PowerShell reported:

```text
NumberOfCores               12
NumberOfLogicalProcessors   24
ThreadCount                 24
```

## Lesson Learned

When observed CPU topology does not match hardware capability, verify the entire configuration chain rather than assuming a hardware fault:

```text
Hardware capability
→ BIOS/UEFI configuration
→ boot configuration
→ operating-system virtualization/security state
→ vendor tuning utilities
→ final OS-visible topology
```

The issue was resolved before enabling the full virtualization platform so VM resource planning could be based on the host's actual available CPU capacity.
