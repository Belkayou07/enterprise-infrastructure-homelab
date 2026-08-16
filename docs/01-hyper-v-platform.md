# Chapter 1 — Hyper-V Platform

## Goal

I am enabling, validating, and organizing Microsoft Hyper-V on my Windows 11 Pro desktop so the host is ready to run the BelkaCorp virtual infrastructure.

## Why This Matters in a Company

A virtualization platform allows multiple isolated operating systems and infrastructure roles to share one physical server. By building this lab myself, I am learning how to manage the host, virtual machines, virtual disks, virtual network adapters, virtual switches, resource allocation, and the difference between the management operating system and guest systems.

## What I Must Understand First

- **Host** — my physical Windows desktop running Hyper-V.
- **Guest / VM** — an operating system running virtually on the host.
- **vCPU** — virtual processors assigned to a VM.
- **VHDX** — Hyper-V virtual hard-disk file format.
- **Virtual NIC** — a network adapter presented to a VM.
- **Virtual switch** — a software Ethernet switch connecting VM NICs to other VMs, the Windows host, or an external network depending on switch type.

## Quick Schema

```text
PHYSICAL WINDOWS HOST
        |
        +-- Hyper-V hypervisor/platform
        +-- Hyper-V Manager / PowerShell
        +-- Virtual switches
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
7. [x] Configure and verify the Windows host's MGMT virtual adapter.
8. [x] Create and pre-boot verify a small test VM.
9. [ ] Boot TEST01 and install Ubuntu Server.
10. [ ] Verify guest CPU, memory, disk, network, and host-to-guest management connectivity.
11. [ ] Complete the remaining Chapter 1 evidence and close the chapter.

## Step 1 — Enable and Verify Hyper-V

I enabled Hyper-V from an elevated PowerShell session:

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

After restarting Windows, I verified:

| Check | Observed state |
|---|---|
| `Microsoft-Hyper-V-All` | Enabled |
| Hyper-V Virtual Machine Management service (`vmms`) | Running, Automatic |
| Hyper-V PowerShell command `Get-VM` | Available from module `Hyper-V` |
| Hyper-V Default Switch | Present, Internal |

### Evidence

![Hyper-V verification](../screenshots/chapter-01/01-01-hyperv-verification.png)

## Step 2 — Hyper-V Storage Convention

My storage inspection showed one usable Windows volume:

| Volume | File system | Size | Free space at inspection |
|---|---|---:|---:|
| `C:` | NTFS | 952.8 GB | 691.2 GB |

I created and configured this dedicated lab structure:

```text
C:\Hyper-V\BelkaCorp\
├── Virtual Machines\
└── Virtual Hard Disks\
```

```text
VirtualMachinePath  C:\Hyper-V\BelkaCorp
VirtualHardDiskPath C:\Hyper-V\BelkaCorp\Virtual Hard Disks
```

### Evidence

![Hyper-V storage configuration](../screenshots/chapter-01/01-02-hyperv-storage-configuration.png)

## Step 3 — Create and Verify Virtual Switches

My planned Hyper-V switches are:

```text
Default Switch -> FW01 WAN
vSW-USERS      -> Private
vSW-SERVERS    -> Private
vSW-MGMT       -> Internal
```

I created the custom switches with `New-VMSwitch`.

### Real Troubleshooting Incident — Duplicate Switch Objects

I accidentally pasted the creation commands more than once, which created duplicate switch objects with the same friendly names. I noticed the problem while using both PowerShell and Hyper-V Virtual Switch Manager. Because no lab VMs were attached yet, I removed the extra objects manually in the GUI and verified the cleaned state again with PowerShell.

I documented the incident in `troubleshooting/002-hyper-v-duplicate-virtual-switches.md`.

My final verified switch inventory was:

| Switch | Count | Type |
|---|---:|---|
| `Default Switch` | 1 | Internal |
| `vSW-MGMT` | 1 | Internal |
| `vSW-SERVERS` | 1 | Private |
| `vSW-USERS` | 1 | Private |

I also verified that each remaining switch has a distinct Hyper-V `Id`.

### Evidence

![Duplicate switches detected](../screenshots/chapter-01/01-03a-duplicate-switches-detected.png)

![Final switch inventory](../screenshots/chapter-01/01-03b-virtual-switches-final.png)

![Virtual Switch Manager](../screenshots/chapter-01/01-03c-virtual-switch-manager-gui.png)

## Step 4 — Windows Host MGMT Adapter

Because `vSW-MGMT` is an Internal switch, Hyper-V created a host-side adapter named `vEthernet (vSW-MGMT)`.

Before configuration I observed:

```text
Status    Up
DHCP      Enabled
IPv4      169.254.112.94/16
```

The `169.254.x.x` address showed that the interface had no usable DHCP lease on the isolated management network.

I then deliberately configured the host management address:

```text
Interface   vEthernet (vSW-MGMT)
IPv4        10.10.30.10/24
DHCP        Disabled
Gateway     none
```

I removed the temporary link-local address, assigned `10.10.30.10/24`, and intentionally did **not** configure a default gateway on this interface so my normal Windows Internet route remains separate from the lab management network.

### Final Verification

I verified:

```text
DHCP              Disabled
ConnectionState   Connected
IPAddress         10.10.30.10
PrefixLength      24
PrefixOrigin      Manual
AddressState      Preferred
Connected route   10.10.30.0/24 -> 0.0.0.0
Default gateway   none
```

### Evidence Status

Captured in the working session; repository upload pending:

- `01-04a-mgmt-static-ip-configuration`
- `01-04b-mgmt-static-ip-verification`

## Step 5 — TEST01 Pre-Boot Validation

I created a small temporary Linux VM to prove that the Hyper-V platform, storage convention, virtual networking, and Generation 2 firmware configuration work together before I deploy the real BelkaCorp infrastructure roles.

### TEST01 Design

```text
TEST01
├── Generation 2
├── 2 vCPU
├── 2 GB startup RAM
├── VHDX: C:\Hyper-V\BelkaCorp\Virtual Hard Disks\TEST01.vhdx
├── vNIC: vSW-MGMT
├── Ubuntu Server ISO: C:\Hyper-V\ISOs\ubuntu-26.04-live-server-amd64.iso
└── Secure Boot: Microsoft UEFI Certificate Authority
```

I used the Hyper-V Manager wizard so I could also understand the GUI workflow, then used PowerShell to verify the resulting VM configuration precisely.

### Pre-Boot PowerShell Verification

I verified the following before starting the VM:

| Property | Verified state |
|---|---|
| Name | `TEST01` |
| State | Off |
| Generation | 2 |
| ProcessorCount | 2 |
| MemoryStartup | 2147483648 bytes (2 GiB) |
| Virtual disk | `C:\Hyper-V\BelkaCorp\Virtual Hard Disks\TEST01.vhdx` |
| Disk controller | SCSI |
| Virtual switch | `vSW-MGMT` |
| Secure Boot | On |
| Secure Boot template | `MicrosoftUEFICertificateAuthority` |

The pre-boot `MacAddress` field displayed `000000000000`. I have not treated that as the final network validation; I will re-check the adapter after TEST01 starts and Hyper-V has an active guest network interface.

### Evidence Status

Captured in the working session; repository upload pending:

- `01-05a-test-vm-wizard-summary`
- `01-05b-test-vm-preboot-verification`

## Evidence Workflow

```text
UNDERSTAND
    |
IMPLEMENT
    |
VERIFY
    |
CAPTURE USEFUL EVIDENCE
    |
DOCUMENT
    |
COMMIT
```

## AI-Assisted Workflow

I use AI as a learning, troubleshooting, and documentation assistant during this project. I still execute and verify the infrastructure changes myself, and I only document states that I have actually observed in the lab.

## Interview Explanation Target

At the end of this chapter, I should be able to explain:

- why I selected Hyper-V for this host;
- host vs guest virtualization;
- vCPU, RAM, VHDX, virtual NICs, and virtual switches;
- External vs Internal vs Private switches;
- why my USERS and SERVERS networks are isolated from the Windows host;
- why the host joins only the MGMT network;
- why the MGMT host adapter has a static address but no default gateway;
- how I detected and corrected the duplicate-switch incident;
- why I used a Generation 2 test VM and how I verified its firmware, storage, CPU, memory, and network attachment before booting it.

## Status

**IN PROGRESS**

Current step: boot `TEST01`, install Ubuntu Server, then verify the guest and host-to-guest management network end to end.
