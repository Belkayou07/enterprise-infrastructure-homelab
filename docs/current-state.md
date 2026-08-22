# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.1 — Pre-Deployment Baseline and OPNsense Installer Verification
- **Current step:** 3.2 — Create and Verify the `FW01` VM Before First Boot
- **Open issues:** None blocking progress; initial VM defaults still require remediation before first boot

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

The OPNsense 26.7 `amd64` DVD installer was downloaded, its compressed image passed SHA-256 verification, and the extracted ISO was placed at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

Verified SHA-256:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

`FW01` has now been created through the Hyper-V wizard and remains powered off.

### Initial FW01 PowerShell audit

Verified actual state:

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

Correct already:

```text
Generation 2
4096 MB fixed memory
VHDX path
OPNsense ISO path
powered-off state
```

Still to change before boot:

```text
12 vCPU                  -> 2 vCPU
Automatic checkpoints   -> Disabled
Secure Boot              -> Disabled
1 NIC                    -> 4 NICs total
```

Planned adapter map:

```text
WAN      -> Default Switch
USERS    -> vSW-USERS
SERVERS  -> vSW-SERVERS
MGMT     -> vSW-MGMT
```

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

The Windows host will keep its normal Wi-Fi default route and will receive specific BelkaCorp routes only after `FW01` is configured and `10.10.30.1` is reachable.

## Evidence Workflow — Chapter 3

The Chapter 3 evidence landing area now exists at:

```text
screenshots/chapter-03/
```

Tracked evidence:

```text
03-01-fw01-predeployment-baseline            CAPTURED, not committed
03-02-opnsense-download-hash-verification    CAPTURED, not committed
03-03-fw01-wizard-summary                    CAPTURED, not committed
03-04-fw01-initial-preboot-audit              next capture
03-05-fw01-final-preboot-verification         planned after remediation
```

The actual PNG must exist in the repository before an item is marked `COMMITTED`.

## Immediate Next Work

While `FW01` is still off:

1. Capture the current initial pre-boot audit as `03-04-fw01-initial-preboot-audit`.
2. Change CPU count to 2.
3. Disable automatic checkpoints.
4. Disable Secure Boot.
5. Keep Dynamic Memory disabled with 4096 MB startup RAM.
6. Keep the existing Default Switch adapter as WAN and add USERS, SERVERS, and MGMT adapters.
7. Run one clean final PowerShell verification and capture `03-05-fw01-final-preboot-verification`.
8. Only then start the OPNsense installer.

Do **not** add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until `FW01` is actually configured and `10.10.30.1` is reachable.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.
