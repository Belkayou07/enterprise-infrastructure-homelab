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
6. [x] Create and verify the planned virtual switches.
7. [ ] Configure the Windows host's MGMT virtual adapter.
8. [ ] Deploy a small test VM.
9. [ ] Verify CPU, memory, disk, and network operation.
10. [ ] Complete evidence/documentation for the remaining Chapter 1 steps.

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

### Evidence

![Hyper-V verification PowerShell evidence](../screenshots/chapter-01/01-01-hyperv-verification.png)

## Step 2 — Inspect Host and Define Storage Convention

The storage inspection showed one usable Windows volume:

| Volume | File system | Size | Free space |
|---|---|---:|---:|
| `C:` | NTFS | 952.8 GB | 691.2 GB |

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

### Evidence

![Hyper-V storage configuration evidence](../screenshots/chapter-01/01-02-hyperv-storage-configuration.png)

**Step 2 is complete.**

## Step 3 — Create and Verify Virtual Switches

The BelkaCorp design requires three lab-side Hyper-V switches in addition to the Windows-managed Default Switch:

```text
Default Switch -> FW01 WAN
vSW-USERS      -> Private
vSW-SERVERS    -> Private
vSW-MGMT       -> Internal
```

Initial creation commands:

```powershell
New-VMSwitch -Name "vSW-USERS" -SwitchType Private
New-VMSwitch -Name "vSW-SERVERS" -SwitchType Private
New-VMSwitch -Name "vSW-MGMT" -SwitchType Internal
```

### Real Troubleshooting Incident — Duplicate Switch Objects

During creation, the commands were pasted more than once. This produced multiple Hyper-V switch objects with the same friendly names.

The duplicate state was noticed while reviewing both PowerShell output and Hyper-V Virtual Switch Manager. The extra custom switch entries were then removed manually in the GUI before any VMs had been attached.

The incident is documented in:

`troubleshooting/002-hyper-v-duplicate-switches.md`

### Final Verification

The cleaned configuration was verified with:

```powershell
Get-VMSwitch |
    Group-Object Name |
    Sort-Object Name |
    Select-Object Name, Count

Get-VMSwitch |
    Sort-Object Name |
    Select-Object Name, SwitchType, Id
```

Final observed state:

| Switch | Count | Type |
|---|---:|---|
| `Default Switch` | 1 | Internal |
| `vSW-MGMT` | 1 | Internal |
| `vSW-SERVERS` | 1 | Private |
| `vSW-USERS` | 1 | Private |

The final inventory also showed a distinct Hyper-V `Id` for each switch.

### Why These Types?

```text
vSW-USERS      Private  -> lab VMs only
vSW-SERVERS    Private  -> lab VMs only
vSW-MGMT       Internal -> lab VMs + Windows host
```

The virtual switch itself does not own the subnet gateway IP. Later, the `FW01` virtual NIC attached to each network will own the `.1` gateway address for that subnet.

### Evidence Status

The real screenshots have been captured in the working session. Planned repository filenames:

- `01-03a-duplicate-switches-detected.png`
- `01-03b-virtual-switches-final.png`
- `01-03c-virtual-switch-manager-gui.png`

They remain evidence-pending until the actual PNG files are uploaded to `screenshots/chapter-01/` and verified in GitHub.

**Switch configuration itself is verified and complete.**

## Step 4 — Windows Host MGMT Adapter

Next, inspect the host-side `vEthernet (vSW-MGMT)` adapter created by the Internal switch and deliberately assign its management-network address. It must not receive a default gateway that could interfere with the Windows host's normal Internet route.

## Evidence Workflow

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

## GitHub Evidence Rule

Do not commit passwords, license keys, Windows product keys, private addresses unrelated to the isolated lab, or unnecessary raw system dumps.

Useful evidence includes validated states, sanitized configuration, real troubleshooting, and selected screenshots that add evidence beyond text alone.

## Interview Explanation Target

At the end of the chapter, be able to explain:

- why Hyper-V was selected for this host;
- host vs guest virtualization;
- vCPU, RAM, VHDX, virtual NICs, and virtual switches;
- External vs Internal vs Private switches;
- how the BelkaCorp VM networks remain isolated while `FW01` performs routing;
- how the physical Windows workstation retains a management path into the lab;
- how the duplicate-switch incident was detected, corrected, and verified.

## Status

**IN PROGRESS**

Current step: commit the Step 3 screenshots, then configure the Windows host's `vEthernet (vSW-MGMT)` adapter.
