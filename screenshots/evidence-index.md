# Evidence Index

This index tracks evidence captured during the real build. Screenshots are only considered repository evidence once the binary file is committed under `screenshots/`.

## Chapter 0 — Host Audit and Troubleshooting

| Evidence | Planned repository path | What it proves | Capture status | Repository status |
|---|---|---|---|---|
| Host CPU overview | `screenshots/chapter-00/00-01-host-cpu.png` | AMD Ryzen 9 7900, 12 cores, 24 logical processors, virtualization enabled | Captured | Binary upload pending |
| SMT remediation verification | `screenshots/chapter-00/00-03-smt-24-logical-processors.png` | Windows reports 12 cores, 24 logical processors, and 24 threads after remediation | Captured | Binary upload pending |

The earlier Ryzen Master screenshot showing `Simultaneous Multithreading = OFF` is also valid troubleshooting evidence. It should be retained if available because it documents the actual root-cause state rather than a recreated condition.

## Chapter 1 — Hyper-V Platform

| Evidence | Planned repository path | What it proves | Capture status | Repository status |
|---|---|---|---|---|
| Hyper-V platform verification | `screenshots/chapter-01/01-01-hyperv-verification.png` | Hyper-V feature enabled, VMMS running/automatic, Hyper-V PowerShell module available, Default Switch present | Captured | Binary upload pending |

## Evidence Rule

Evidence should document the real engineering journey:

```text
OBSERVE / BUILD
      |
      v
VERIFY
      |
      v
CAPTURE USEFUL EVIDENCE
      |
      v
DOCUMENT WHAT IT PROVES
      |
      v
COMMIT
```

Do not recreate failures merely to obtain screenshots. Real troubleshooting evidence is preferred over staged evidence.
