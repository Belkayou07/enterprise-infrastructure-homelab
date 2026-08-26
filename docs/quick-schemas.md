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
   3.5     Configure internal gateways  [DONE]
   3.6     Verify MGMT + routing        [NOW]
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

## Applied OPNsense Roles and Addresses

```text
OPNsense role   NIC   BelkaCorp role   IPv4
-------------------------------------------------
WAN             hn0   WAN              DHCP
LAN             hn3   MGMT             10.10.30.1/24
OPT1            hn1   USERS            10.10.10.1/24
OPT2            hn2   SERVERS          10.10.20.1/24
```

MGMT verification:

```text
Windows host 10.10.30.10/24
        |
        | ping 10.10.30.1 -> 4/4 replies
        | HTTPS Web GUI   -> reachable
        v
FW01 LAN / hn3
10.10.30.1/24
```

## Routing Foundation to Verify

```text
Expected connected routes on FW01
10.10.10.0/24 -> hn1 / USERS
10.10.20.0/24 -> hn2 / SERVERS
10.10.30.0/24 -> hn3 / MGMT

Expected default route
0.0.0.0/0 -> WAN / hn0 upstream
```

These connected routes should be created automatically because FW01 owns an IP address inside each subnet. Step 3.6 verifies that OPNsense actually installed them before Windows-specific routes are added.

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
                  10.10.30.1/24
                     [OK]
```

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

## BelkaCorp Addressing

```text
10.10.0.0/16
|
+-- USERS    10.10.10.0/24   gateway 10.10.10.1 [CONFIGURED]
|      DHCP later: 10.10.10.100 - 10.10.10.199
|
+-- SERVERS  10.10.20.0/24   gateway 10.10.20.1 [CONFIGURED]
|      DC01  10.10.20.10
|      LNX01 10.10.20.20
|      MON01 10.10.20.30
|
+-- MGMT     10.10.30.0/24   gateway 10.10.30.1 [CONFIGURED + VERIFIED]
       Windows host 10.10.30.10
       TEST01       10.10.30.20
```

## Windows Host Routing Model

```text
Windows normal default route
0.0.0.0/0 -> home Wi-Fi gateway

BelkaCorp routes later
10.10.10.0/24 -> 10.10.30.1
10.10.20.0/24 -> 10.10.30.1
```

The specific Windows routes remain deferred until Step 3.6 verifies FW01's own routing table.

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
