# Quick Schemas

Compact visual notes for my homelab. These are intentionally simplified and should be readable in a few seconds. Detailed explanations belong in the chapter documentation.

## Project Progress

```text
CHAPTER 0  Design / host audit          [DONE]
   0.1     Hardware audit               [DONE]
   0.2     Virtualization investigation [DONE]
   0.3     CPU / SMT troubleshooting    [DONE]
   0.4     Enterprise architecture      [DONE]

CHAPTER 1  Hyper-V platform             [DONE]
   1.1     Enable + verify Hyper-V      [DONE]
   1.2     Host storage convention      [DONE]
   1.3     Virtual switches             [DONE]
   1.3b    Host MGMT adapter            [DONE]
   1.4     TEST01 pre-boot config       [DONE]
   1.5     Ubuntu install / first boot  [DONE]
   1.6     TEST01 static MGMT network   [DONE]
   1.7     Runtime + reboot validation  [DONE]
   1.8     Final Chapter 1 audit        [DONE]

CHAPTER 2  Network architecture         [DONE]
   2.1A    Windows network baseline     [DONE]
   2.1B    TEST01 network baseline      [DONE]
   2.2     Subnet + address allocation  [DONE]
   2.3     L2 / L3 boundaries           [DONE]
   2.4     Gateway + routing behavior   [DONE]
   2.5     DHCP + DNS ownership         [DONE]
   2.6     Implementation plan          [DONE]
   2.7     Final Chapter 2 audit        [DONE]

CHAPTER 3  Firewall / routing           [NOW]
   3.1     Baseline + installer verify  [DONE]
   3.2     FW01 pre-boot configuration  [NOW]
   3.3     Install OPNsense
   3.4     Map four FW01 interfaces
   3.5     Configure internal gateways
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
OPNsense 26.7 amd64 installer  SHA-256 verified [OK]
Installer ISO                  C:\Hyper-V\ISOs\OPNsense-26.7-dvd-amd64.iso
FW01 wizard                    completed
FW01 first boot                NOT YET

Initial PowerShell audit:
State                          Off
Generation                     2
Memory                         4096 MB fixed [OK]
vCPU                           12 -> change to 2
Automatic checkpoints          Enabled -> disable
Secure Boot                    Enabled -> disable
Network adapters               1 -> add 3 internal adapters
Initial NIC                    Default Switch -> WAN
VHDX                           C:\Hyper-V\BelkaCorp\Virtual Hard Disks\FW01.vhdx [OK]
```

Next checkpoint: remediate the four pre-boot defaults, verify all four NIC attachments, capture/store `03-06-fw01-final-preboot-verification.png`, update documentation, and only then start OPNsense.

## Chapter 1 Flow

```text
ENABLE HYPER-V          [DONE]
      |
      v
RESTART WINDOWS         [DONE]
      |
      v
VERIFY FEATURE + TOOLS  [DONE]
      |
      v
SET VM STORAGE          [DONE]
      |
      v
CREATE VIRTUAL SWITCHES [DONE]
      |
      v
CONFIGURE HOST MGMT     [DONE]
      |
      v
CREATE TEST01           [DONE]
      |
      v
INSTALL + BOOT UBUNTU   [DONE]
      |
      v
CONFIGURE TEST01 MGMT   [DONE]
      |
      v
VERIFY HOST <-> GUEST   [DONE]
      |
      v
VERIFY RUNTIME + REBOOT [DONE]
      |
      v
FINAL CHAPTER AUDIT     [DONE]
      |
      v
CHAPTER 1 COMPLETE      [DONE]
```

## Chapter 2 Pre-Router Baseline

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal

Same-subnet route       direct / on-link [OK]
Windows -> TEST01       reachable        [OK]
TEST01 -> Windows       4/4              [OK]
ARP / neighbor mapping  learned          [OK]
Route to 10.10.20.20    unreachable      [EXPECTED]
FW01                    not deployed yet
```

The MGMT subnet works directly at Layer 2. Neither host currently has a BelkaCorp route to the future SERVERS or USERS networks.

## Hyper-V Verified State

```text
Microsoft-Hyper-V-All  -> Enabled
vmms service           -> Running / Automatic
Get-VM                 -> Hyper-V module available
Default Switch         -> Present / Internal
```

## Final Hyper-V Storage

```text
C:\Hyper-V\BelkaCorp\
|
+-- Virtual Machines\
|
+-- Virtual Hard Disks\
```

```text
VirtualMachinePath  -> C:\Hyper-V\BelkaCorp
VirtualHardDiskPath -> C:\Hyper-V\BelkaCorp\Virtual Hard Disks
```

## Final Hyper-V Switch Inventory

```text
Default Switch  Internal  count 1
vSW-MGMT        Internal  count 1
vSW-SERVERS     Private   count 1
vSW-USERS       Private   count 1
```

I accidentally created duplicate custom switches when I ran `New-VMSwitch` commands multiple times. I detected them through PowerShell and Virtual Switch Manager, removed the extras manually in the GUI, and then re-verified the final counts and unique switch IDs with PowerShell.

## Host MGMT Adapter

```text
Windows host
    |
    | 10.10.30.10/24
    | DHCP disabled
    | NO default gateway
    v
vEthernet (vSW-MGMT)
    |
    v
vSW-MGMT  Internal
    |
    +---- FW01 MGMT 10.10.30.1/24 later
```

## TEST01 Final MGMT Network

```text
Windows host                  TEST01
10.10.30.10/24                10.10.30.20/24
      |                              |
      +--------- vSW-MGMT -----------+
                  Internal

Ping Windows -> TEST01   [OK]
Ping TEST01  -> Windows  [OK]
Default gateway          [NONE yet]
```

Both hosts are in `10.10.30.0/24`, so they communicate directly through the Layer-2 virtual switch. A router is not required for this same-subnet test.

## One-Way Ping Troubleshooting

```text
Windows -> TEST01   OK
TEST01  -> Windows  FAIL
          |
          v
Windows MGMT profile = Public
Windows Firewall      = enabled
          |
          v
Add narrow inbound rule:
ICMPv4 Echo
interface = vEthernet (vSW-MGMT)
local     = 10.10.30.10
remote    = 10.10.30.0/24
          |
          v
Ping both ways        OK
```

## TEST01 Model

```text
TEST01
|
+-- Generation 2
+-- 2 vCPU
+-- Dynamic Memory
|     startup  = 2048 MB
|     minimum  = 1536 MB
|     maximum  = 4096 MB
|     buffer   = 20%
+-- TEST01.vhdx -> C:\Hyper-V\BelkaCorp\Virtual Hard Disks\
+-- vNIC        -> vSW-MGMT
+-- Ubuntu 26.04 LTS installed
+-- eth0        -> 10.10.30.20/24
+-- Secure Boot -> Microsoft UEFI Certificate Authority
+-- Automatic checkpoints -> Disabled
+-- Existing checkpoints  -> None
```

## Dynamic Memory Quick Note

```text
STARTUP RAM
RAM available to start/boot the VM
        |
        v
DYNAMIC MEMORY ENABLED
        |
        +-- MINIMUM = lowest Hyper-V should reclaim toward
        |
        +-- DEMAND  = what the guest currently needs
        |
        +-- ASSIGNED = what Hyper-V currently gives the VM
        |
        +-- MAXIMUM = upper growth limit
```

TEST01 incident:

```text
Old minimum = 512 MB
        |
        v
Hyper-V reclaimed RAM aggressively
        |
        v
Guest showed very low memory
        |
        v
Inspect Get-VMMemory + Get-VM
        |
        v
New limits = 2048 startup / 1536 min / 4096 max
        |
        v
Cold boot
        |
        v
Assigned memory = 1536 MB [OK]
Network persistence           [OK]
```

## Checkpoint Quick Note

```text
BASE DISK
TEST01.vhdx
    |
    v
CHECKPOINT CREATED
    |
    +--> writes move into .avhdx differencing disk
    |
    v
MORE CHECKPOINTS
    |
    +--> .avhdx chain can grow
    |
    v
REMOVE CHECKPOINT THROUGH HYPER-V
    |
    v
Hyper-V merges checkpoint data
    |
    v
TEST01.vhdx active again
```

Final TEST01 checkpoint state:

```text
Automatic checkpoints  Disabled
Existing checkpoints   None
Active disk             TEST01.vhdx
TEST01 .avhdx files     None
Post-cleanup ping       4/4 [OK]
```

## Physical Model

```text
+----------------------------------------------------------+
|                  WINDOWS 11 DESKTOP                      |
|                                                          |
|  Normal workstation                Lab host              |
|  - Browser                         - Hyper-V             |
|  - PowerShell                      - Virtual networks     |
|  - VS Code                         - VMs                 |
|  - Git/GitHub                      - GNS3 later          |
+----------------------------------------------------------+
```

## Final Enterprise / Hyper-V Model

```text
                         INTERNET
                            |
                    Hyper-V Default Switch
                            |
                          WAN NIC
                           [FW01]
                          OPNsense
        +-------------------+-------------------+
        |                   |                   |
   10.10.10.1          10.10.20.1          10.10.30.1
     USERS               SERVERS                MGMT
        |                   |                   |
   vSW-USERS          vSW-SERVERS           vSW-MGMT
    Private              Private              Internal
        |                   |                   |
   [CLIENT01]       +-------+-------+       Windows host
   DHCP later       |       |       |       10.10.30.10
                  [DC01]  [LNX01] [MON01]
                   .10     .20      .30
```

## FW01 Interfaces

```text
FW01 / OPNsense
|
+-- NIC 1  WAN      -> Hyper-V Default Switch -> Internet
+-- NIC 2  USERS    -> vSW-USERS    -> 10.10.10.1/24
+-- NIC 3  SERVERS  -> vSW-SERVERS  -> 10.10.20.1/24
+-- NIC 4  MGMT     -> vSW-MGMT     -> 10.10.30.1/24
```

## Hyper-V Switch Type Quick Note

```text
EXTERNAL
VMs + host <-> physical network adapter / external network

INTERNAL
VMs <-> Windows host
No direct physical-network connection

PRIVATE
VMs <-> VMs only
Windows host cannot directly join that switch
```

## IP Addressing

```text
BELKACORP PRIVATE SPACE
10.10.0.0/16
    |
    +-- USERS    10.10.10.0/24   gateway .1
    |      DHCP: 10.10.10.100 - 10.10.10.199
    |
    +-- SERVERS  10.10.20.0/24   gateway .1
    |      DC01  10.10.20.10
    |      LNX01 10.10.20.20
    |      MON01 10.10.20.30
    |
    +-- MGMT     10.10.30.0/24   gateway .1 later
           Windows host 10.10.30.10
           TEST01       10.10.30.20
```

## Routing Between Two Subnets

```text
CLIENT01 10.10.10.125/24
       |
       | destination 10.10.20.10 is NOT local
       v
Default gateway 10.10.10.1
       |
       v
      FW01
       |
       | route: 10.10.20.0/24 is directly connected
       | firewall policy must allow it
       v
SERVERS interface 10.10.20.1
       |
       v
DC01 10.10.20.10/24
```

## DHCP and DNS Ownership

```text
CLIENT01
  DHCP client
     |
     | broadcast on USERS
     v
FW01
  DHCP relay
     |
     v
DC01 10.10.20.10
  DHCP server
  DNS server
  AD DS later
```

```text
USERS DHCP scope
10.10.10.100 - 10.10.10.199
Gateway -> 10.10.10.1
DNS     -> 10.10.20.10
```

Domain clients use `DC01` for internal DNS so Active Directory service records can be resolved. Public names can be forwarded or resolved upstream by `DC01` later.

## Chapter 2 Implementation Order

```text
FW01 deploy
   |
   v
Identify 4 NICs
   |
   v
Configure .1 gateways
   |
   v
Verify MGMT access
   |
   v
Add Windows BelkaCorp routes
   |
   v
Deploy DC01
   |
   v
AD DS + internal DNS
   |
   v
Windows Server DHCP
   |
   v
FW01 DHCP relay
   |
   v
Deploy CLIENT01
   |
   v
End-to-end validation
```

Do not add the Windows routes before `FW01` exists: `10.10.30.1` must first be a real, reachable next hop.

Quick note:

```text
10.10.0.0/16 = reserved BelkaCorp address block / supernet
It is NOT a router or a host.
```

## /24 Quick Note

```text
Example: 10.10.20.10/24

Network     10.10.20.0
Gateway     10.10.20.1   (our design)
Host        10.10.20.10
Broadcast   10.10.20.255
```

## Server Roles

```text
FW01      Firewall / routing / policy
DC01      Active Directory + DNS
CLIENT01  Domain user workstation
LNX01     Linux administration / services
MON01     Monitoring / observability
TEST01    Temporary Hyper-V platform validation VM
```

## Naming Rule

```text
ROLE + NUMBER

FW01
DC01
LNX01
MON01
CLIENT01
TEST01
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
   +---- wrong ----> next hypothesis
   |
 correct
   v
FIX
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

I will keep updating this file as the architecture changes and as new chapters introduce important concepts.