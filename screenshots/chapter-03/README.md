# Chapter 3 Evidence Queue

This folder is the landing area for real screenshots from Chapter 3 — Firewall and Routing.

## Evidence states

- `PLANNED` — the screenshot has been identified as useful evidence but has not been captured yet.
- `CAPTURED` — the screenshot exists locally / in the chat but is not yet present in this repository folder.
- `COMMITTED` — the actual PNG exists in this folder and has been verified.

## Current queue

| Evidence | State | What it proves |
|---|---|---|
| [`03-01-fw01-predeployment-baseline.png`](03-01-fw01-predeployment-baseline.png) | COMMITTED | PowerShell baseline showing no existing `FW01`, the four expected Hyper-V switches, and the ISO directory before the OPNsense image was added. |
| [`03-02-opnsense-download-hash-verification.png`](03-02-opnsense-download-hash-verification.png) | COMMITTED | PowerShell SHA-256 verification of the downloaded OPNsense 26.7 `amd64` DVD image. |
| [`03-03-opnsense-iso-staged.png`](03-03-opnsense-iso-staged.png) | COMMITTED | File Explorer showing the extracted OPNsense ISO staged in `C:\Hyper-V\ISOs` beside the existing Ubuntu installer. |
| [`03-04-fw01-wizard-summary.png`](03-04-fw01-wizard-summary.png) | COMMITTED | Hyper-V wizard summary showing `FW01`, Generation 2, 4096 MB, Default Switch, the dedicated VHDX path, and the OPNsense ISO. |
| [`03-05-fw01-initial-preboot-audit.png`](03-05-fw01-initial-preboot-audit.png) | COMMITTED | PowerShell audit showing the initial VM defaults before remediation: 12 vCPU, automatic checkpoints enabled, Secure Boot enabled, and only one network adapter. |
| [`03-06-fw01-final-preboot-verification.png`](03-06-fw01-final-preboot-verification.png) | COMMITTED | Final PowerShell verification showing `FW01` powered off with 2 vCPU, fixed 4096 MB RAM, Secure Boot off, automatic checkpoints off, correct VHDX/ISO paths, and four correctly attached NICs. |
| [`03-07-opnsense-boot-menu.png`](03-07-opnsense-boot-menu.png) | COMMITTED | Shows `FW01` running and successfully booting the OPNsense installation media to the OPNsense boot menu. |
| [`03-08-fw01-pxe-after-install.png`](03-08-fw01-pxe-after-install.png) | COMMITTED | Shows the first post-install start falling through to PXE instead of booting the installed VHDX, creating the boot-order troubleshooting checkpoint. |
| [`03-09-opnsense-installed-first-disk-boot.png`](03-09-opnsense-installed-first-disk-boot.png) | COMMITTED | Shows OPNsense 26.7 successfully booted from the installed `FW01.vhdx`, with the normal console menu available after the firmware boot-order correction. |
| [`03-10-fw01-hyperv-nic-mac-map.png`](03-10-fw01-hyperv-nic-mac-map.png) | COMMITTED | PowerShell maps the four named Hyper-V adapters to their switches and assigned MAC addresses, establishing the authoritative host-side reference for OPNsense interface mapping. |
| [`03-11-opnsense-guest-nic-mac-map.png`](03-11-opnsense-guest-nic-mac-map.png) | COMMITTED | OPNsense `ifconfig` output maps `hn0` through `hn3` to the same four MAC addresses, proving the guest-side interface identity before role assignment. |

The PXE incident is documented in `../../troubleshooting/006-fw01-post-install-pxe-boot.md`.

## Working rule

For Chapter 3 I finish evidence and documentation at the same checkpoint as the technical work instead of postponing it until later.

```text
DO WORK
   |
   v
VERIFY
   |
   v
CAPTURE EVIDENCE IF USEFUL
   |
   v
STORE + VERIFY IN GITHUB
   |
   v
UPDATE DOCUMENTATION
   |
   v
ONLY THEN CONTINUE
```

I only keep screenshots that prove a meaningful implementation, verification, or troubleshooting state. I do not recreate historical failures just for evidence.