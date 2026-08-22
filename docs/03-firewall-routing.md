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

I downloaded the OPNsense 26.7 DVD image for `amd64` and verified the compressed installer with SHA-256 before extracting it. The calculated hash matched the expected release hash:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

I then extracted the ISO and placed it at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

This gives me a verified installation source before I create or boot the firewall VM.

### Evidence status

The pre-deployment baseline and installer-hash screenshots have been captured locally but are not yet committed to the repository.

## 3.2 — FW01 VM Creation and Pre-Boot Verification

I used the Hyper-V New Virtual Machine Wizard to prepare the initial `FW01` VM shell with the following settings:

```text
Name             FW01
Generation       2
Startup memory   4096 MB
Initial network  Default Switch
Virtual disk     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
Disk type        dynamically expanding VHDX
Installer        C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

The wizard summary confirms those values before creation. The VM was then completed through the wizard, but I have not booted it yet.

Before first boot I still need to verify and finalize the VM configuration:

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

I will not assign the Windows host's BelkaCorp routes yet because `10.10.30.1` is not a verified reachable next hop until OPNsense is installed and the MGMT interface is configured.

### Evidence status

The Hyper-V wizard summary screenshot has been captured and inspected. It is not yet committed to the repository.

## Current Position

`FW01` has been created through the Hyper-V wizard and remains unbooted. The next step is to verify the actual VM object and apply the remaining pre-boot settings before starting OPNsense for the first time.
