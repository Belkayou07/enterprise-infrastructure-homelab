# Chapter 1 — Hyper-V Platform

## Goal

Enable, validate, and organize Microsoft Hyper-V on the Windows 11 Pro desktop so the host is ready to run the BelkaCorp virtual infrastructure.

## Why This Matters in a Company

A virtualization platform allows multiple isolated operating systems and infrastructure roles to share one physical server. Administrators need to understand the host, virtual machines, virtual disks, virtual network adapters, virtual switches, resource allocation, and the difference between the management operating system and guest systems.

## What Must Be Understood First

Before building VMs, understand these terms:

- **Host** — the physical Windows desktop running Hyper-V.
- **Guest / VM** — an operating system running virtually on the host.
- **vCPU** — virtual processors assigned to a VM; they consume scheduling time on the physical CPU rather than representing dedicated physical cores.
- **VHDX** — Hyper-V virtual hard-disk file format.
- **Virtual NIC** — a network adapter presented to a VM.
- **Virtual switch** — a software Ethernet switch connecting VM NICs to other VMs, the Windows host, or an external network depending on switch type.

## Quick Schema

```text
PHYSICAL WINDOWS HOST
        |
        +-- Hyper-V hypervisor/platform
        |
        +-- Hyper-V Manager / PowerShell
        |
        +-- Virtual switches
        |
        +-- Virtual machines
               |
               +-- vCPU
               +-- RAM
               +-- VHDX
               +-- virtual NIC(s)
```

## Chapter Plan

1. Enable the full Hyper-V Windows feature.
2. Restart Windows.
3. Verify the Hyper-V feature, management tools, and hypervisor state.
4. Inspect the Hyper-V host configuration.
5. Define VM and VHDX storage conventions.
6. Create the planned virtual switches.
7. Configure the Windows host's MGMT virtual adapter.
8. Deploy a small test VM.
9. Verify CPU, memory, disk, and network operation.
10. Document evidence and troubleshooting.

## Step 1 — Enable Hyper-V

Run an elevated PowerShell session and execute:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

A restart is expected before the platform is considered installed and verified.

### Verification After Restart

Do not mark Step 1 complete until the host reports the actual post-restart state. Planned checks include:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
Get-Command Get-VM
Get-VMSwitch
```

The exact observed output will be documented after execution.

## Planned Virtual Networking

```text
Default Switch -> FW01 WAN
vSW-USERS      -> Private
vSW-SERVERS    -> Private
vSW-MGMT       -> Internal
```

These switches will not be created until the Hyper-V platform itself has been verified.

## GitHub Evidence Rule

Do not commit passwords, license keys, Windows product keys, private addresses unrelated to the isolated lab, or unnecessary raw system dumps.

Useful evidence includes:

- validated feature state;
- sanitized Hyper-V host/switch inventory;
- final VM/switch conventions;
- troubleshooting records for real issues encountered;
- selected screenshots only when they add value beyond text/configuration.

## Interview Explanation Target

At the end of the chapter, be able to explain:

- why Hyper-V was selected for this host;
- host vs guest virtualization;
- vCPU, RAM, VHDX, virtual NICs, and virtual switches;
- External vs Internal vs Private switches;
- how the BelkaCorp VM networks remain isolated while `FW01` performs routing;
- how the physical Windows workstation retains a management path into the lab.

## Status

**IN PROGRESS**

Current step: enable Hyper-V, restart, and verify the resulting platform state.
