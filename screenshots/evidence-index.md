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

I documented the duplicate-switch incident in `../troubleshooting/002-hyper-v-duplicate-virtual-switches.md` and the one-way ICMP incident in `../troubleshooting/003-windows-firewall-blocked-test01-icmp.md`.

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
