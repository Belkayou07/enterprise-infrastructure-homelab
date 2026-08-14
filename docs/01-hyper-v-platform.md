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

1. [x] Enable the full Hyper-V Windows feature.
2. [x] Restart Windows.
3. [x] Verify the Hyper-V feature, management tools, and hypervisor state.
4. [x] Inspect the Hyper-V host configuration.
5. [x] Define VM and VHDX storage conventions.
6. [ ] Create the planned virtual switches.
7. [ ] Configure the Windows host's MGMT virtual adapter.
8. [ ] Deploy a small test VM.
9. [ ] Verify CPU, memory, disk, and network operation.
10. [ ] Document evidence and troubleshooting.

## Step 1 — Enable and Verify Hyper-V

Hyper-V was enabled from an elevated PowerShell session with:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Windows was restarted before validation.

### Verified Post-Restart State

Observed on 2026-08-14:

| Check | Observed state |
|---|---|
| `Microsoft-Hyper-V-All` | Enabled |
| Hyper-V Virtual Machine Management service (`vmms`) | Running, Automatic |
| Hyper-V PowerShell command `Get-VM` | Available from module `Hyper-V` |
| Hyper-V Default Switch | Present, Internal |

Verification commands used:

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All |
    Select-Object FeatureName, State

Get-Service vmms |
    Select-Object Name, Status, StartType

Get-Command Get-VM |
    Select-Object Name, Source

Get-VMSwitch |
    Select-Object Name, SwitchType
```

This confirms that the feature, management service, PowerShell module, and Hyper-V virtual-switch subsystem are all operational.

### Evidence

![Hyper-V verification PowerShell evidence](../screenshots/chapter-01/01-01-hyperv-verification.png)

The screenshot above is the real post-restart verification from this host. It shows Hyper-V enabled, `vmms` running automatically, the Hyper-V PowerShell module available, and the Default Switch present as an Internal switch.

Related Chapter 0 host evidence is stored under `screenshots/chapter-00/`, and the complete inventory is tracked in [`screenshots/evidence-index.md`](../screenshots/evidence-index.md).

## Step 2 — Inspect Host and Define Storage Convention

The storage inspection showed one usable Windows volume:

| Volume | File system | Size | Free space |
|---|---|---:|---:|
| `C:` | NTFS | 952.8 GB | 691.2 GB |

The original Hyper-V defaults were:

```text
VirtualMachinePath  C:\ProgramData\Microsoft\Windows\Hyper-V
VirtualHardDiskPath C:\ProgramData\Microsoft\Windows\Virtual Hard Disks
```

These defaults were functional but less convenient for a deliberate portfolio lab. A dedicated BelkaCorp location was therefore created and configured.

### Final Storage Convention

```text
C:\Hyper-V\BelkaCorp\
├── Virtual Machines\
└── Virtual Hard Disks\
```

Hyper-V host defaults were configured as:

```text
VirtualMachinePath  C:\Hyper-V\BelkaCorp
VirtualHardDiskPath C:\Hyper-V\BelkaCorp\Virtual Hard Disks
```

The `Virtual Machines` and `Virtual Hard Disks` directories were verified to exist after configuration.

Commands used:

```powershell
$base = "C:\Hyper-V\BelkaCorp"
$vhd  = "C:\Hyper-V\BelkaCorp\Virtual Hard Disks"

New-Item -ItemType Directory -Path "$base\Virtual Machines" -Force | Out-Null
New-Item -ItemType Directory -Path $vhd -Force | Out-Null

Set-VMHost `
    -VirtualMachinePath $base `
    -VirtualHardDiskPath $vhd

Get-VMHost |
    Select-Object VirtualMachinePath, VirtualHardDiskPath

Get-ChildItem "C:\Hyper-V\BelkaCorp" |
    Select-Object Name, FullName
```

### Why This Layout?

The goal is not to imply that Hyper-V requires this exact structure. The purpose is to make the lab's VM assets predictable, easy to find, easier to back up later, and clearly separated from Microsoft's generic default locations.

### Evidence Status

Configuration is verified. Screenshot evidence `01-02-hyperv-storage-configuration.png` is the remaining evidence checkpoint for this step.

## Planned Virtual Networking

```text
Default Switch -> FW01 WAN
vSW-USERS      -> Private
vSW-SERVERS    -> Private
vSW-MGMT       -> Internal
```

The enterprise switches will be created after the storage evidence checkpoint is committed.

## Evidence Workflow

Every meaningful implementation stage follows this sequence:

```text
UNDERSTAND
    |
    v
IMPLEMENT
    |
    v
VERIFY
    |
    v
CAPTURE USEFUL EVIDENCE
    |
    v
DOCUMENT
    |
    v
COMMIT
```

Screenshots should prove a real result, configuration, or troubleshooting event. Routine commands with no portfolio value do not need screenshots.

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

Current step: commit the verified storage evidence, then create the BelkaCorp Hyper-V virtual switches.
