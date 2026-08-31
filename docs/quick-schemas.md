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
   3.6     Verify MGMT + routing        [DONE]
   3.7     Add Windows lab routes       [DONE]
   3.8     Validate routing / policy    [NOW]
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

## Verified FW01 Routing Foundation

```text
10.10.10.0/24 -> hn1 / USERS    [OK]
10.10.20.0/24 -> hn2 / SERVERS  [OK]
10.10.30.0/24 -> hn3 / MGMT     [OK]
default       -> hn0 / WAN      [OK]
```

Explicit route lookups:

```text
10.10.10.50 -> hn1
10.10.20.50 -> hn2
10.10.30.10 -> hn3
```

Connectivity from FW01:

```text
10.10.30.10 -> 4/4 replies, 0% loss
1.1.1.1     -> 4/4 replies, 0% loss
```

## Verified Windows Host Routing

```text
Windows normal Internet route
0.0.0.0/0       -> 192.168.0.1 -> Wi-Fi

Persistent BelkaCorp routes
10.10.10.0/24   -> 10.10.30.1 -> vEthernet (vSW-MGMT)
10.10.20.0/24   -> 10.10.30.1 -> vEthernet (vSW-MGMT)
```

Actual route-selection checks:

```text
10.10.10.50 -> vSW-MGMT -> 10.10.30.1
10.10.20.50 -> vSW-MGMT -> 10.10.30.1
1.1.1.1     -> Wi-Fi    -> 192.168.0.1
```

So Windows sends BelkaCorp USERS/SERVERS traffic to FW01 while normal Internet traffic remains on Wi-Fi.

## Current Network Model

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

## Step 3.8 Principle

```text
ROUTE EXISTS
     |
     v
PACKET REACHES FW01
     |
     v
FIREWALL POLICY CHECK
     |
     +--> ALLOW -> forward
     |
     +--> BLOCK -> stop/log
```

A correct route does not automatically mean cross-subnet traffic is permitted. Step 3.8 validates forwarding and firewall policy separately.

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
