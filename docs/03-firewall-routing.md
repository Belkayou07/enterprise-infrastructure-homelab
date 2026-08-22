# Chapter 3 — Firewall and Routing

## Goal

I am deploying `FW01` as the Layer-3 router and firewall for the BelkaCorp lab. The immediate objective is to turn the three isolated Hyper-V networks from Chapter 2 into routed networks while preserving clear security boundaries and keeping the Windows host's normal Internet route separate from the lab.

## Why This Matters in a Company

A firewall/router is the control point between network segments. It must have the correct interfaces, addresses, routes, and policy before servers and clients can reliably communicate across subnets. I am therefore building and verifying `FW01` in stages rather than assigning routes or deploying later services before the router actually exists.

## Chapter 3 Plan

- [x] 3.1 — Verify the pre-deployment Hyper-V baseline and OPNsense installer
- [ ] 3.2 — Create and verify the `FW01` VM before first boot
- [ ] 3.3 — Install OPNsense
- [ ] 3.4 — Identify and map the four interfaces
- [ ] 3.5 — Configure USERS, SERVERS, and MGMT gateway interfaces
- [ ] 3.6 — Verify management connectivity and routing foundation
- [ ] 3.7 — Add the Windows host's specific BelkaCorp routes
- [ ] 3.8 — Validate routed traffic and initial firewall behavior
- [ ] 3.9 — Complete the Chapter 3 audit

## 3.1 — Pre-Deployment Baseline and Installer Verification

Before creating the firewall VM, I verified that no existing `FW01` VM was present and confirmed the expected Hyper-V switch inventory:

```text
Default Switch  Internal
vSW-MGMT        Internal
vSW-SERVERS     Private
vSW-USERS       Private
```

At that point, the Hyper-V ISO directory contained only the existing Ubuntu Server installer, so the OPNsense installer still had to be added.

I downloaded the OPNsense 26.7 DVD image for `amd64` and verified the compressed installer with SHA-256 before extraction. The calculated hash matched the expected release hash:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

I then extracted the ISO and staged it at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

### Evidence

- [`03-01-fw01-predeployment-baseline.png`](../screenshots/chapter-03/03-01-fw01-predeployment-baseline.png) — pre-deployment VM/switch/ISO baseline.
- [`03-02-opnsense-download-hash-verification.png`](../screenshots/chapter-03/03-02-opnsense-download-hash-verification.png) — SHA-256 verification of the downloaded installer.
- [`03-03-opnsense-iso-staged.png`](../screenshots/chapter-03/03-03-opnsense-iso-staged.png) — extracted OPNsense ISO present in the Hyper-V ISO directory.

## 3.2 — FW01 VM Creation and Pre-Boot Verification

I used the Hyper-V New Virtual Machine Wizard to create the initial `FW01` VM shell with:

```text
Name             FW01
Generation       2
Startup memory   4096 MB
Initial network  Default Switch
Virtual disk     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
Disk type        dynamically expanding VHDX
Installer        C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

The VM was created through the wizard and remains unbooted.

### Wizard evidence

[`03-04-fw01-wizard-summary.png`](../screenshots/chapter-03/03-04-fw01-wizard-summary.png) records the wizard summary before creation.

### Initial PowerShell audit

I then inspected the actual Hyper-V VM object instead of assuming the wizard defaults matched the intended firewall design.

Verified initial state:

```text
State                    Off
Generation               2
Automatic checkpoints    Enabled
vCPU                     12
Dynamic Memory           Disabled
Startup memory           4096 MB
Secure Boot              Enabled
VHDX                     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
DVD                       C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
Network adapters          1
Initial switch            Default Switch
```

The fixed 4096 MB memory policy, virtual disk path, installer path, VM generation, and powered-off state are already correct. The displayed minimum/maximum memory values are not active limits because Dynamic Memory is disabled.

The audit exposed four settings that must be corrected before first boot:

```text
12 vCPU                  -> 2 vCPU
Automatic checkpoints   -> Disabled
Secure Boot              -> Disabled
1 network adapter        -> 4 total adapters
```

The current Default Switch adapter will become the WAN adapter. I still need to add:

```text
USERS    -> vSW-USERS
SERVERS  -> vSW-SERVERS
MGMT     -> vSW-MGMT
```

### Initial audit evidence

[`03-05-fw01-initial-preboot-audit.png`](../screenshots/chapter-03/03-05-fw01-initial-preboot-audit.png) records the actual pre-remediation state discovered in PowerShell.

### Intended final pre-boot state

```text
vCPU                    2
Memory                  4096 MB fixed
Secure Boot             Disabled
Automatic checkpoints   Disabled
WAN NIC                  Default Switch
USERS NIC                vSW-USERS
SERVERS NIC              vSW-SERVERS
MGMT NIC                 vSW-MGMT
```

The next evidence item will be `03-06-fw01-final-preboot-verification.png`, but only after those changes have been applied and verified.

I will not add the Windows host's routes to `10.10.10.0/24` or `10.10.20.0/24` yet because `10.10.30.1` is not a real, verified next hop until OPNsense is installed and the MGMT interface is configured.

## Evidence and Documentation Workflow

From this point forward, I complete repository work at the same meaningful checkpoint as the technical work:

```text
IMPLEMENT
   |
   v
VERIFY
   |
   v
CAPTURE USEFUL EVIDENCE
   |
   v
STORE + VERIFY IN GITHUB
   |
   v
UPDATE DOCUMENTATION
   |
   v
CONTINUE
```

This prevents the repository from lagging behind the real infrastructure state.

## Current Position

`FW01` exists, remains powered off, and has never booted. The current task is still **3.2**: remediate the initial VM defaults, add the three missing internal network adapters, run a clean final pre-boot verification, store that evidence, and only then proceed to the OPNsense installation.