# Chapter 3 Evidence Queue

This folder is the landing area for real screenshots from Chapter 3 — Firewall and Routing.

## Evidence states

- `PLANNED` — the screenshot has been identified as useful evidence but has not been captured yet.
- `CAPTURED` — the screenshot exists locally / in the chat but is not yet present in this repository folder.
- `COMMITTED` — the actual PNG exists in this folder and has been verified.

## Current queue

| Evidence | State | Expected content |
|---|---|---|
| `03-01-fw01-predeployment-baseline.png` | CAPTURED | PowerShell baseline showing no existing `FW01`, the four expected Hyper-V switches, and the ISO directory before the OPNsense image was added. |
| `03-02-opnsense-download-hash-verification.png` | CAPTURED | PowerShell SHA-256 verification of the downloaded OPNsense 26.7 `amd64` DVD image. |
| `03-03-fw01-wizard-summary.png` | CAPTURED | Hyper-V wizard summary showing `FW01`, Generation 2, 4096 MB, Default Switch, the VHDX path, and OPNsense ISO. |
| `03-04-fw01-initial-preboot-audit.png` | PLANNED | PowerShell audit showing the initial VM defaults that still need correction: 12 vCPU, automatic checkpoints enabled, Secure Boot enabled, and only one network adapter. |
| `03-05-fw01-final-preboot-verification.png` | PLANNED | Final PowerShell verification after remediation, showing 2 vCPU, fixed 4096 MB RAM, Secure Boot off, automatic checkpoints off, and four correctly attached NICs. |

## Working rule

For Chapter 3, I capture evidence at the point where it proves a meaningful implementation or troubleshooting state. I do not batch screenshots until the end of the chapter. After each useful screenshot is captured, I move it into this folder, verify the actual image, and only then mark it `COMMITTED` in `../evidence-index.md`.
