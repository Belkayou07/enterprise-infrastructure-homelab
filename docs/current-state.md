# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.2 — Create and Verify the `FW01` VM Before First Boot
- **Current step:** 3.3 — Install OPNsense
- **Open issue:** First post-install boot fell through to Hyper-V PXE instead of starting OPNsense from `FW01.vhdx`; firmware boot order has now been corrected for a controlled retry.

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

The OPNsense 26.7 `amd64` DVD installer was downloaded and the compressed image passed SHA-256 verification.

Verified SHA-256:

```text
95CAFEDDA6D5B22CE832E249DC2309110FBEE19F813AD78CF28BB3D387186BFB
```

The extracted installer remains staged at:

```text
C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
```

### Final verified FW01 pre-boot state

```text
Generation               2
Automatic checkpoints    Disabled
vCPU                     2
Dynamic Memory           Disabled
Startup memory           4096 MB
Secure Boot              Off
VHDX                     C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx

WAN      -> Default Switch
USERS    -> vSW-USERS
SERVERS  -> vSW-SERVERS
MGMT     -> vSW-MGMT
```

### OPNsense installation state

`FW01` successfully booted from the OPNsense ISO, reached the live console, and completed installation to the dedicated virtual disk.

Verified progress:

```text
OPNsense boot menu       Reached successfully
Live console login       Reached successfully
Installer                Completed
Filesystem               ZFS
Disk layout              Single-disk stripe
Install target           da0 / FW01.vhdx
Root password            Set
Post-install state       VM halted
Mounted DVD ISO paths    None
Installed VHDX           C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
```

### Current boot troubleshooting checkpoint

The first post-install start did not boot OPNsense and instead reached Hyper-V `Start PXE over IPv4`.

Investigation verified:

```text
FW01.vhdx attachment     SCSI controller 0, location 0 [OK]
Secure Boot              Off [OK]
Initial firmware order   mixed Drive / Network entries
```

The hard disk has now been explicitly set as the first boot device. Detailed verification shows:

```text
1. HardDiskDrive — SCSI 0:0 (FW01.vhdx)
2. DVD Drive — SCSI 0:1
3. WAN network adapter
4. USERS network adapter
5. SERVERS network adapter
6. MGMT network adapter
7. DVD Drive — SCSI 0:2
```

The next action is to start `FW01` again and determine whether the corrected firmware order resolves the boot failure. If it still falls through to PXE, the next investigation will focus on the installed UEFI/bootloader state rather than reinstalling blindly.

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
03-09-opnsense-installed-first-disk-boot.png       PLANNED
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

Continue **3.3 — Install OPNsense** by starting `FW01` with `FW01.vhdx` explicitly first in the Hyper-V firmware boot order. If the installed OPNsense console appears, capture the successful disk boot as `03-09-opnsense-installed-first-disk-boot.png` and close the incident. If PXE appears again, continue bootloader/UEFI diagnosis before considering reinstallation.

Do not begin interface mapping or add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until the installed firewall has booted successfully and the later MGMT interface is configured and reachable at `10.10.30.1`.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.