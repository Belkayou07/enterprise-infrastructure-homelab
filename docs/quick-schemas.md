# Quick Schemas

Compact visual notes for my homelab. Detailed implementation, troubleshooting, and evidence belong in the chapter documentation.

## Project Progress

```text
CHAPTER 0  Design / host audit          [DONE]
CHAPTER 1  Hyper-V platform             [DONE]
CHAPTER 2  Network architecture         [DONE]

CHAPTER 3  Firewall / routing           [NOW]
   3.1     Baseline + installer verify  [DONE]
   3.2     FW01 pre-boot configuration  [DONE]
   3.3     Install OPNsense             [DONE]
   3.4     Map four FW01 interfaces     [DONE]
   3.5     Configure internal gateways  [NOW]
   3.6     Verify MGMT + routing
   3.7     Add Windows lab routes
   3.8     Validate routing / policy
   3.9     Final Chapter 3 audit

CHAPTER 4  Windows Server / AD
CHAPTER 5  Linux administration
CHAPTER 6  Segmentation / access control
CHAPTER 7  Security hardening
CHAPTER 8  Monitoring
CHAPTER 9  Backup / disaster recovery
CHAPTER 10 Automation
CHAPTER 11 Cloud / DevOps extension
```

## Chapter 3 Current State

```text
FW01 / OPNsense 26.7
Generation                     2
Memory                         4096 MB fixed
vCPU                           2
Automatic checkpoints          Disabled
Secure Boot                    Off
Installed VHDX                 C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx
Installer ISO                  Detached
Installed disk boot            [OK]
```

The first post-install boot fell through to PXE. I verified the disk and firmware state, placed `FW01.vhdx` first in the Generation 2 boot order, and then verified a successful installed-system boot. The incident is documented in `../troubleshooting/006-fw01-post-install-pxe-boot.md`.

## Verified FW01 Interface Identity

```text
Hyper-V role   Switch           MAC                  OPNsense NIC
-----------------------------------------------------------------
WAN            Default Switch   00:15:5D:38:01:05    hn0
USERS          vSW-USERS        00:15:5D:38:01:06    hn1
SERVERS        vSW-SERVERS      00:15:5D:38:01:07    hn2
MGMT           vSW-MGMT         00:15:5D:38:01:08    hn3
```

## Applied OPNsense Roles

```text
OPNsense role   NIC   BelkaCorp role
-------------------------------------
WAN             hn0   WAN
LAN             hn3   MGMT
OPT1            hn1   USERS
OPT2            hn2   SERVERS
```

Current console state immediately after role assignment:

```text
LAN  / hn3   192.168.1.1/24      temporary OPNsense default
OPT1 / hn1   no BelkaCorp IP yet
OPT2 / hn2   no BelkaCorp IP yet
WAN  / hn0   DHCP 172.25.218.248/20
```

The WAN DHCP lease confirms the WAN adapter is connected to the Hyper-V Default Switch. Full Internet connectivity still requires separate verification.

## Next Gateway Configuration

```text
LAN  / hn3 / MGMT      -> 10.10.30.1/24   [NEXT]
OPT1 / hn1 / USERS     -> 10.10.10.1/24
OPT2 / hn2 / SERVERS   -> 10.10.20.1/24
WAN  / hn0             -> DHCP via Default Switch
```

MGMT is configured first because the Windows host is already connected to `vSW-MGMT` at `10.10.30.10/24`.

## Current MGMT Path

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal
                         |
                     hn3 / LAN
                       FW01
                  10.10.30.1/24 NEXT
```

Before `10.10.30.1` exists, Windows and TEST01 still communicate directly with each other on the same Layer-2 subnet.

## Final Enterprise Model

```text
                         INTERNET
                            |
                    Hyper-V Default Switch
                            |
                          hn0 / WAN
                           [FW01]
                          OPNsense
        +-------------------+-------------------+
        |                   |                   |
   hn1 / USERS         hn2 / SERVERS       hn3 / MGMT
   10.10.10.1          10.10.20.1          10.10.30.1
        |                   |                   |
   vSW-USERS          vSW-SERVERS           vSW-MGMT
    Private              Private              Internal
        |                   |                   |
   [CLIENT01]       +-------+-------+       Windows host
   DHCP later       |       |       |       10.10.30.10
                  [DC01]  [LNX01] [MON01]
                   .10     .20      .30
```

## Hyper-V Switch Types

```text
EXTERNAL
VMs + host <-> physical network / external network

INTERNAL
VMs <-> Windows host
No direct physical-network connection

PRIVATE
VMs <-> VMs only
Windows host cannot directly join that switch
```

## BelkaCorp Addressing

```text
10.10.0.0/16
|
+-- USERS    10.10.10.0/24   gateway 10.10.10.1
|      DHCP later: 10.10.10.100 - 10.10.10.199
|
+-- SERVERS  10.10.20.0/24   gateway 10.10.20.1
|      DC01  10.10.20.10
|      LNX01 10.10.20.20
|      MON01 10.10.20.30
|
+-- MGMT     10.10.30.0/24   gateway 10.10.30.1
       Windows host 10.10.30.10
       TEST01       10.10.30.20
```

## Cross-Subnet Routing Model

```text
CLIENT01 10.10.10.x
       |
       | destination 10.10.20.10 is outside local /24
       v
10.10.10.1 / FW01 USERS
       |
       | routing decision + firewall policy
       v
10.10.20.1 / FW01 SERVERS
       |
       v
DC01 10.10.20.10
```

Routing decides where traffic can be forwarded. Firewall policy separately decides whether that traffic is allowed.

## Windows Host Routing Model

```text
Windows normal default route
0.0.0.0/0 -> home Wi-Fi gateway

BelkaCorp routes later
10.10.10.0/24 -> 10.10.30.1
10.10.20.0/24 -> 10.10.30.1
```

The specific BelkaCorp routes are not added until `10.10.30.1` is configured and reachable. More-specific `/24` routes will then take precedence over the normal default route for lab traffic.

## DHCP and DNS Ownership

```text
CLIENT01
  DHCP client
     |
     | broadcast on USERS
     v
FW01
  DHCP relay later
     |
     v
DC01 10.10.20.10
  Windows DHCP
  DNS
  AD DS later
```

```text
USERS DHCP scope later
10.10.10.100 - 10.10.10.199
Gateway -> 10.10.10.1
DNS     -> 10.10.20.10
```

## Server Roles

```text
FW01      Firewall / routing / policy
DC01      Active Directory + DNS + DHCP later
CLIENT01  Domain user workstation
LNX01     Linux administration / services
MON01     Monitoring / observability
TEST01    Temporary Hyper-V validation VM
```

## Troubleshooting Method

```text
SYMPTOM
   |
   v
OBSERVE
   |
   v
FORM HYPOTHESIS
   |
   v
TEST ONE LAYER
   |
   v
FIX ONE VARIABLE
   |
   v
VERIFY
   |
   v
DOCUMENT
```

## Learning Rule

```text
UNDERSTAND MANUALLY
        |
        v
BUILD MANUALLY
        |
        v
TROUBLESHOOT
        |
        v
DOCUMENT
        |
        v
AUTOMATE
```
