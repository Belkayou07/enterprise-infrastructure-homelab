# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.1 — Pre-Deployment Baseline and OPNsense Installer Verification
- **Current step:** 3.2 — Create and Verify the `FW01` VM Before First Boot
- **Open issues:** None currently blocking progress

## Verified Live State

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal
```

Verified behavior:

- Windows and TEST01 communicate directly inside `10.10.30.0/24`.
- TEST01 has a directly connected route for `10.10.30.0/24`.
- TEST01 resolves the Windows host at Layer 2 through neighbor/ARP discovery.
- A route lookup toward `10.10.20.20` remains unreachable until `FW01` is configured.

## Chapter 3 Deployment State

The pre-deployment baseline confirmed:

```text
FW01 VM          absent before creation
Default Switch   Internal
vSW-MGMT         Internal
vSW-SERVERS      Private
vSW-USERS        Private
```

The OPNsense 26.7 `amd64` DVD installer was downloaded, its compressed image passed SHA-256 verification, and the extracted ISO was placed at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

The verified SHA-256 was:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

The Hyper-V wizard summary for `FW01` has now been captured and inspected. It shows:

```text
Name             FW01
Generation       2
Memory           4096 MB
Network          Default Switch
Virtual disk     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
Installer        C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

The wizard was completed, but `FW01` has not been booted yet. The actual VM object and remaining pre-boot settings still need to be verified.

## Completed Network Design

BelkaCorp allocation block:

```text
10.10.0.0/16
```

Current subnets:

```text
USERS    10.10.10.0/24   planned gateway 10.10.10.1
SERVERS  10.10.20.0/24   planned gateway 10.10.20.1
MGMT     10.10.30.0/24   planned gateway 10.10.30.1
```

Planned addressing:

```text
USERS
10.10.10.1          FW01
10.10.10.100-.199   planned DHCP range

SERVERS
10.10.20.1          FW01
10.10.20.10         DC01
10.10.20.20         LNX01
10.10.20.30         MON01

MGMT
10.10.30.1          FW01
10.10.30.10         Windows Hyper-V host
10.10.30.20         TEST01 temporary validation VM
```

## Layer and Routing Model

Each custom Hyper-V switch is a distinct Layer-2 segment:

```text
vSW-USERS    -> USERS   -> 10.10.10.0/24
vSW-SERVERS  -> SERVERS -> 10.10.20.0/24
vSW-MGMT     -> MGMT    -> 10.10.30.0/24
```

`FW01` will connect to all three internal segments and route between them. Normal hosts use the `FW01` address on their own subnet as gateway. The Windows host will keep its Wi-Fi default route and will receive specific BelkaCorp routes only after `FW01` exists and `10.10.30.1` is reachable.

## Planned FW01 Interfaces

```text
FW01 / OPNsense
|
+-- NIC 1  WAN      -> Hyper-V Default Switch
+-- NIC 2  USERS    -> vSW-USERS    -> 10.10.10.1/24
+-- NIC 3  SERVERS  -> vSW-SERVERS  -> 10.10.20.1/24
+-- NIC 4  MGMT     -> vSW-MGMT     -> 10.10.30.1/24
```

Before first boot, verify/finalize:

```text
2 vCPU
4096 MB fixed memory
Secure Boot disabled
Automatic checkpoints disabled
all four NIC attachments correct
```

## Evidence Status

Chapter 3 screenshots currently captured but not yet committed:

```text
03-01-fw01-predeployment-baseline
03-02-opnsense-download-hash-verification
03-03-fw01-wizard-summary
```

Do not mark these as committed until the actual image files exist in the repository.

## Immediate Next Work

Verify the newly created `FW01` object in PowerShell and apply the remaining pre-boot configuration. Do not start OPNsense until CPU, memory policy, Secure Boot, checkpoint policy, virtual disk, ISO, and all four NIC attachments have been checked.

Do **not** add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until `FW01` is actually configured and `10.10.30.1` is reachable.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.
