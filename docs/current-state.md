# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.3 — Install OPNsense
- **Current step:** 3.4 — Identify and map the four interfaces
- **Open issues:** None currently blocking progress

## Verified Live State

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal
                         |
                       FW01
                  OPNsense 26.7
                     running
```

Verified behavior:

- Windows and TEST01 communicate directly inside `10.10.30.0/24`.
- TEST01 has a directly connected route for `10.10.30.0/24`.
- TEST01 resolves the Windows host at Layer 2 through neighbor/ARP discovery.
- A route lookup toward `10.10.20.20` remains unreachable until `FW01` receives the planned internal gateway addresses and routing/firewall policy is configured.

## Chapter 3 Deployment State

The OPNsense 26.7 `amd64` DVD installer was downloaded and the compressed image passed SHA-256 verification.

Verified SHA-256:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

The extracted installer remains staged at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

### Verified FW01 platform state

```text
Generation               2
Automatic checkpoints    Disabled
vCPU                     2
Dynamic Memory           Disabled
Startup memory           4096 MB
Secure Boot              Off
VHDX                     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
Mounted DVD ISO paths    None

WAN      -> Default Switch
USERS    -> vSW-USERS
SERVERS  -> vSW-SERVERS
MGMT     -> vSW-MGMT
```

### OPNsense installation state

`FW01` successfully booted from the verified OPNsense ISO, reached the live console, and completed installation to the dedicated virtual disk using ZFS on the single virtual disk.

The first post-install start fell through to Hyper-V PXE. Investigation confirmed the VHDX was attached correctly and Secure Boot was off. I explicitly placed the hard disk first in the Generation 2 firmware boot order, then retried the boot.

Final verification now shows:

```text
Installer ISO             detached
FW01.vhdx                 attached on SCSI 0:0
FW01.vhdx boot priority   first
Installed OPNsense boot   successful
Root login                successful
Normal console menu       reached
```

The PXE incident is closed in `troubleshooting/006-fw01-post-install-pxe-boot.md`.

### Current OPNsense interface state

The successful installed console currently shows OPNsense's automatic/default assignments:

```text
LAN  -> hn0 -> 192.168.1.1/24
WAN  -> hn1
```

These assignments are **not yet accepted as the BelkaCorp interface design**. All four Hyper-V adapters must first be matched to the guest interfaces by MAC address. No BelkaCorp `.1` gateway addresses have been configured yet.

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

The Windows host keeps its normal Wi-Fi default route. Specific routes to `10.10.10.0/24` and `10.10.20.0/24` are only added after `FW01` is configured and `10.10.30.1` is reachable.

## Evidence Status

The current Chapter 3 evidence is committed and verified under `screenshots/chapter-03/`:

```text
03-01-fw01-predeployment-baseline.png             COMMITTED
03-02-opnsense-download-hash-verification.png     COMMITTED
03-03-opnsense-iso-staged.png                     COMMITTED
03-04-fw01-wizard-summary.png                     COMMITTED
03-05-fw01-initial-preboot-audit.png               COMMITTED
03-06-fw01-final-preboot-verification.png          COMMITTED
03-07-opnsense-boot-menu.png                       COMMITTED
03-08-fw01-pxe-after-install.png                   COMMITTED
03-09-opnsense-installed-first-disk-boot.png       COMMITTED
```

## Working Rule

For each meaningful checkpoint:

```text
IMPLEMENT
   |
   v
VERIFY
   |
   v
CAPTURE EVIDENCE IF USEFUL
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

Do not batch evidence or documentation until the end of the chapter.

## Immediate Next Work

Continue **3.4 — Identify and map the four interfaces**.

First, capture the authoritative Hyper-V adapter names, switch connections, and assigned MAC addresses with PowerShell. Then compare those MAC addresses to the OPNsense guest interfaces before changing any interface assignment.

Do **not** configure the planned `.1` gateway addresses or add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until the interface mapping is verified.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.