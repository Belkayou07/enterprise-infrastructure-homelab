# Evidence Index

This index tracks evidence captured from the real BelkaCorp homelab build. Evidence is only marked complete when the actual file exists in the repository.

## Status Legend

- `CAPTURED` — screenshot was taken but is not yet stored in the repository.
- `COMMITTED` — the actual evidence file is present in GitHub.

## Chapter 0 — Host Audit and Design

| Evidence | Status | What it proves |
|---|---|---|
| [`chapter-00/00-01-host-cpu.png`](chapter-00/00-01-host-cpu.png) | COMMITTED | Windows Task Manager reports AMD Ryzen 9 7900, 12 cores, 24 logical processors, and hardware virtualization enabled. |
| [`chapter-00/00-03-smt-24-logical-processors.png`](chapter-00/00-03-smt-24-logical-processors.png) | COMMITTED | PowerShell verification after remediation reports 12 cores, 24 logical processors, and 24 threads. |

The SMT troubleshooting record is documented separately in `../troubleshooting/001-ryzen-smt-logical-processors.md`.

## Chapter 1 — Hyper-V Platform

| Evidence | Status | What it proves |
|---|---|---|
| [`chapter-01/01-01-hyperv-verification.png`](chapter-01/01-01-hyperv-verification.png) | COMMITTED | Hyper-V is enabled; VMMS is running automatically; the Hyper-V PowerShell module is available; and the Default Switch is present as an Internal switch. |
| [`chapter-01/01-02-hyperv-storage-configuration.png`](chapter-01/01-02-hyperv-storage-configuration.png) | COMMITTED | Hyper-V host defaults point to the dedicated BelkaCorp storage path and the `Virtual Machines` / `Virtual Hard Disks` directories exist. |

## Evidence Rule

Useful evidence should show a meaningful state or verification result, not every command typed during the project.

The standard workflow is:

```text
UNDERSTAND
   |
IMPLEMENT
   |
VERIFY
   |
CAPTURE EVIDENCE
   |
DOCUMENT
   |
COMMIT
```

Sensitive data, credentials, license keys, unrelated personal information, and unnecessary raw dumps must not be committed.
