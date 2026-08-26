# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.5 — Configure USERS, SERVERS, and MGMT gateway interfaces
- **Current step:** 3.6 — Verify management connectivity and routing foundation
- **Open issues:** None currently blocking progress

## Verified Live State

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal
                         |
                    LAN / hn3
                       FW01
                  10.10.30.1/24
                  OPNsense 26.7
                     running
```

Verified behavior:

- Windows and TEST01 communicate directly inside `10.10.30.0/24`.
- FW01 MGMT is configured as `10.10.30.1/24` on `LAN / hn3`.
- The Windows host at `10.10.30.10/24` successfully pings `10.10.30.1` with 0% packet loss.
- The OPNsense HTTPS Web GUI is reachable from the Windows host at `https://10.10.30.1`.
- USERS is configured as `10.10.10.1/24` on `OPT1 / hn1`.
- SERVERS is configured as `10.10.20.1/24` on `OPT2 / hn2`.

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

Final verification shows:

```text
Installer ISO             detached
FW01.vhdx                 attached on SCSI 0:0
FW01.vhdx boot priority   first
Installed OPNsense boot   successful
Root login                successful
Normal console menu       reached
```

The PXE incident is closed in `troubleshooting/006-fw01-post-install-pxe-boot.md`.

### Verified interface identity and applied roles

The host-side Hyper-V adapter map is:

```text
WAN      Default Switch   00:15:5D:38:01:05
USERS    vSW-USERS        00:15:5D:38:01:06
SERVERS  vSW-SERVERS      00:15:5D:38:01:07
MGMT     vSW-MGMT         00:15:5D:38:01:08
```

The OPNsense guest-side MAC map is:

```text
hn0   00:15:5D:38:01:05
hn1   00:15:5D:38:01:06
hn2   00:15:5D:38:01:07
hn3   00:15:5D:38:01:08
```

The resulting exact mapping is:

```text
WAN      -> hn0 -> Default Switch
USERS    -> hn1 -> vSW-USERS
SERVERS  -> hn2 -> vSW-SERVERS
MGMT     -> hn3 -> vSW-MGMT
```

I applied the OPNsense roles as:

```text
WAN   = hn0
LAN   = hn3   [BelkaCorp MGMT]
OPT1  = hn1   [BelkaCorp USERS]
OPT2  = hn2   [BelkaCorp SERVERS]
```

### Current interface addressing

```text
WAN  / hn0              DHCP from Hyper-V Default Switch
LAN  / hn3 / MGMT       10.10.30.1/24   [CONFIGURED + VERIFIED]
OPT1 / hn1 / USERS      10.10.10.1/24   [CONFIGURED]
OPT2 / hn2 / SERVERS    10.10.20.1/24   [CONFIGURED]
```

The WAN DHCP lease is dynamic and may change. The three BelkaCorp internal gateway addresses are static.

No upstream gateway is configured on the internal interfaces. OPNsense DHCP remains disabled there because the long-term design assigns DHCP ownership to `DC01`.

## Completed Network Design

BelkaCorp allocation block:

```text
10.10.0.0/16
```

Current subnets:

```text
USERS    10.10.10.0/24   gateway 10.10.10.1 [CONFIGURED]
SERVERS  10.10.20.0/24   gateway 10.10.20.1 [CONFIGURED]
MGMT     10.10.30.0/24   gateway 10.10.30.1 [CONFIGURED + VERIFIED]
```

The Windows host keeps its normal Wi-Fi default route. Specific routes to `10.10.10.0/24` and `10.10.20.0/24` are still deferred until Step 3.6 confirms FW01's routing foundation.

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
03-10-fw01-hyperv-nic-mac-map.png                  COMMITTED
03-11-opnsense-guest-nic-mac-map.png               COMMITTED
03-12-opnsense-interface-role-assignment.png       COMMITTED
03-13a-opnsense-mgmt-ip-configured.png             COMMITTED
03-13b-mgmt-connectivity-and-webgui.png             COMMITTED
03-14-opnsense-internal-gateways-configured.png    COMMITTED
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

Continue **3.6 — Verify management connectivity and routing foundation**.

Inspect the OPNsense IPv4 routing table and verify that the three BelkaCorp `/24` networks are directly connected through the correct interfaces, while a WAN/default route remains available.

Do not add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until this routing-table verification is complete.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.