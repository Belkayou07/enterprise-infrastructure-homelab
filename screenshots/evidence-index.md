# Evidence Index

This index tracks evidence captured from my real BelkaCorp homelab build. I only mark evidence complete when the actual file exists in the repository.

## Status Legend

- `CAPTURED` — I took the screenshot but have not yet stored it in the repository.
- `COMMITTED` — the actual evidence file is present in GitHub.

## Chapter 0 — Host Audit and Design

| Evidence | Status | What it proves |
|---|---|---|
| [`chapter-00/00-01-host-cpu.png`](chapter-00/00-01-host-cpu.png) | COMMITTED | Windows Task Manager reports AMD Ryzen 9 7900, 12 cores, 24 logical processors, and hardware virtualization enabled. |
| [`chapter-00/00-03-smt-24-logical-processors.png`](chapter-00/00-03-smt-24-logical-processors.png) | COMMITTED | PowerShell verification after remediation reports 12 cores, 24 logical processors, and 24 threads. |

I documented the SMT troubleshooting record separately in `../troubleshooting/001-ryzen-smt-logical-processors.md`.

## Chapter 1 — Hyper-V Platform

| Evidence | Status | What it proves |
|---|---|---|
| [`chapter-01/01-01-hyperv-verification.png`](chapter-01/01-01-hyperv-verification.png) | COMMITTED | Hyper-V is enabled; VMMS is running automatically; the Hyper-V PowerShell module is available; and the Default Switch is present as an Internal switch. |
| [`chapter-01/01-02-hyperv-storage-configuration.png`](chapter-01/01-02-hyperv-storage-configuration.png) | COMMITTED | Hyper-V host defaults point to the dedicated BelkaCorp storage path and the `Virtual Machines` / `Virtual Hard Disks` directories exist. |
| [`chapter-01/01-03a-duplicate-switches-detected.png`](chapter-01/01-03a-duplicate-switches-detected.png) | COMMITTED | Shows the accidental duplicate custom switch state I detected during verification. |
| [`chapter-01/01-03b-virtual-switches-final.png`](chapter-01/01-03b-virtual-switches-final.png) | COMMITTED | Shows my final PowerShell switch counts, intended switch types, and unique switch IDs after cleanup. |
| [`chapter-01/01-03c-virtual-switch-manager-gui.png`](chapter-01/01-03c-virtual-switch-manager-gui.png) | COMMITTED | Shows the BelkaCorp virtual-switch layout in Hyper-V Virtual Switch Manager. |
| [`chapter-01/01-04a-mgmt-static-ip-configuration.png`](chapter-01/01-04a-mgmt-static-ip-configuration.png) | COMMITTED | Shows me disabling DHCP, removing the temporary link-local address, and assigning `10.10.30.10/24` to the host MGMT adapter. |
| [`chapter-01/01-04b-mgmt-static-ip-verification.png`](chapter-01/01-04b-mgmt-static-ip-verification.png) | COMMITTED | Shows the final MGMT interface state: DHCP disabled, static manual address, connected `/24` route, and no default gateway. |
| [`chapter-01/01-05a-test-vm-wizard-summary.png`](chapter-01/01-05a-test-vm-wizard-summary.png) | COMMITTED | Shows the Hyper-V wizard summary for `TEST01`: Generation 2, 2048 MB, `vSW-MGMT`, dedicated VHDX path, and Ubuntu Server ISO. |
| [`chapter-01/01-05b-test-vm-preboot-verification.png`](chapter-01/01-05b-test-vm-preboot-verification.png) | COMMITTED | Shows PowerShell verification of `TEST01` before boot: 2 vCPU, 2 GiB startup RAM, SCSI VHDX, `vSW-MGMT`, and Linux-compatible Secure Boot template. |
| [`chapter-01/01-06a-test-vm-ubuntu-first-boot.png`](chapter-01/01-06a-test-vm-ubuntu-first-boot.png) | COMMITTED | Shows `TEST01` running in Hyper-V and a successful Ubuntu 26.04 LTS login after installation. |
| [`chapter-01/01-06b-test-vm-network-preconfig.png`](chapter-01/01-06b-test-vm-network-preconfig.png) | COMMITTED | Shows the detected `eth0` interface, its Hyper-V MAC address, no IPv4 address before configuration, and the installer-created Netplan file. |
| [`chapter-01/01-06c-test-vm-static-network-one-way-ping.png`](chapter-01/01-06c-test-vm-static-network-one-way-ping.png) | COMMITTED | Shows `TEST01` configured as `10.10.30.20/24`, its connected route, and the initial failed ping from TEST01 to the Windows host. |
| [`chapter-01/01-06d-test-vm-bidirectional-ping.png`](chapter-01/01-06d-test-vm-bidirectional-ping.png) | COMMITTED | Shows the scoped Windows Firewall ICMP rule and successful ping in both directions between `10.10.30.10` and `10.10.30.20`. |
| [`chapter-01/01-07a-test-vm-runtime-low-memory-detected.png`](chapter-01/01-07a-test-vm-runtime-low-memory-detected.png) | COMMITTED | Shows the guest CPU, disk, network, and connectivity checks while exposing the unexpectedly low guest-visible memory value. |
| [`chapter-01/01-07b-test-vm-dynamic-memory-diagnosis.png`](chapter-01/01-07b-test-vm-dynamic-memory-diagnosis.png) | COMMITTED | Shows Dynamic Memory enabled with 2048 MB startup RAM, a 512 MB minimum, and live assigned/demand values that explained the low guest memory. |
| [`chapter-01/01-07c-test-vm-dynamic-memory-corrected.png`](chapter-01/01-07c-test-vm-dynamic-memory-corrected.png) | COMMITTED | Shows the corrected Dynamic Memory limits: 2048 MB startup, 1536 MB minimum, 4096 MB maximum, and 20% buffer. |
| [`chapter-01/01-07d-test-vm-post-memory-fix-validation.png`](chapter-01/01-07d-test-vm-post-memory-fix-validation.png) | COMMITTED | Shows TEST01 after a cold boot with 2 vCPU, persistent `10.10.30.20/24`, the connected route, and successful host connectivity. |
| [`chapter-01/01-07e-test-vm-dynamic-memory-runtime.png`](chapter-01/01-07e-test-vm-dynamic-memory-runtime.png) | COMMITTED | Shows Hyper-V assigning the corrected 1536 MB minimum at runtime while preserving the 2048/1536/4096 MB Dynamic Memory limits. |
| [`chapter-01/01-08a-hyperv-final-host-audit.png`](chapter-01/01-08a-hyperv-final-host-audit.png) | COMMITTED | Shows the final Hyper-V host audit: feature/service state, BelkaCorp storage defaults, final virtual-switch inventory, and MGMT adapter state. |
| [`chapter-01/01-08b-test01-final-audit.png`](chapter-01/01-08b-test01-final-audit.png) | COMMITTED | Shows TEST01's final VM, Dynamic Memory, storage, network, and connectivity audit and exposes automatic checkpoints as still enabled. |
| [`chapter-01/01-08c-test01-checkpoint-diagnosis.png`](chapter-01/01-08c-test01-checkpoint-diagnosis.png) | COMMITTED | Shows `10.10.30.10/24`, automatic checkpoints enabled, two existing checkpoints, and the active `.avhdx` differencing-disk chain. |
| [`chapter-01/01-08d-test01-checkpoint-cleanup.png`](chapter-01/01-08d-test01-checkpoint-cleanup.png) | COMMITTED | Shows TEST01 powered off, automatic checkpoints disabled, no remaining checkpoints, direct `TEST01.vhdx` attachment, and no remaining TEST01 `.avhdx` files. |
| [`chapter-01/01-08e-test01-final-post-cleanup-validation.png`](chapter-01/01-08e-test01-final-post-cleanup-validation.png) | COMMITTED | Shows TEST01 running after cleanup with automatic checkpoints still disabled, no checkpoints, direct base-VHDX attachment, and successful 4/4 connectivity to `10.10.30.20`. |

I documented the duplicate-switch incident in `../troubleshooting/002-hyper-v-duplicate-virtual-switches.md`, the one-way ICMP incident in `../troubleshooting/003-windows-firewall-blocked-test01-icmp.md`, the TEST01 Dynamic Memory incident in `../troubleshooting/004-test01-dynamic-memory-minimum.md`, and the automatic-checkpoint chain in `../troubleshooting/005-test01-automatic-checkpoint-chain.md`.

## Chapter 2 — Network Architecture and IP Design

| Evidence | Status | What it proves |
|---|---|---|
| [`chapter-02/02-01a-network-baseline-host.png`](chapter-02/02-01a-network-baseline-host.png) | COMMITTED | Shows the pre-router Windows network baseline: final virtual-switch inventory, manual `10.10.30.10/24` MGMT addressing, the directly connected `10.10.30.0/24` route, and the normal Windows default route remaining on Wi-Fi through `192.168.0.1`. |
| [`chapter-02/02-01b-network-baseline-test01.png`](chapter-02/02-01b-network-baseline-test01.png) | COMMITTED | Shows TEST01 at `10.10.30.20/24`, its directly connected MGMT route, direct route selection to `10.10.30.10`, successful 4/4 host connectivity, the learned Windows MAC neighbor, and the expected unreachable result for `10.10.20.20` before a router exists. |

## Chapter 3 — Firewall and Routing

| Evidence | Status | What it proves |
|---|---|---|
| `chapter-03/03-01-fw01-predeployment-baseline.png` | CAPTURED | Shows that `FW01` did not exist before deployment, confirms the expected Hyper-V switch inventory, and shows that the OPNsense installer had not yet been added to the ISO directory. |
| `chapter-03/03-02-opnsense-download-hash-verification.png` | CAPTURED | Shows the SHA-256 verification of the downloaded OPNsense 26.7 `amd64` DVD image before extraction. |
| `chapter-03/03-03-fw01-wizard-summary.png` | CAPTURED | Shows the `FW01` wizard summary with Generation 2, 4096 MB startup memory, Default Switch, dedicated VHDX path, and the OPNsense 26.7 installer ISO. |

These Chapter 3 files remain `CAPTURED` until the actual image files are uploaded to the repository.

## Evidence Rule

I keep useful evidence that shows a meaningful state or verification result rather than capturing every command I type.

My standard workflow is:

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

I do not commit sensitive data, credentials, license keys, unrelated personal information, or unnecessary raw dumps.
