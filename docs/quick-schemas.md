# Quick Schemas

Compact visual notes for the homelab. These are intentionally simplified and should be readable in a few seconds. Detailed explanations belong in the chapter documentation.

## Project Progress

```text
CHAPTER 0  Design / host audit          [IN PROGRESS]
   0.1     Hardware audit               [DONE]
   0.2     Virtualization investigation [DONE]
   0.3     CPU / SMT troubleshooting     [DONE]
   0.4     Enterprise architecture       [NOW]

CHAPTER 1  Hyper-V platform             [NEXT]
CHAPTER 2  Network architecture
CHAPTER 3  Firewall / routing
CHAPTER 4  Windows Server / AD
CHAPTER 5  Linux administration
CHAPTER 6  Segmentation / access control
CHAPTER 7  Security hardening
CHAPTER 8  Monitoring
CHAPTER 9  Backup / disaster recovery
CHAPTER 10 Automation
CHAPTER 11 Cloud / DevOps extension
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

## Enterprise Model

```text
                              INTERNET
                                 |
                              [FW01]
                                 |
             +-------------------+-------------------+
             |                   |                   |
           USERS               SERVERS              MGMT
       10.10.10.0/24       10.10.20.0/24       10.10.30.0/24
       GW 10.10.10.1       GW 10.10.20.1       GW 10.10.30.1
             |                   |                   |
        [CLIENT01]         +-----+-----+        Admin access
        DHCP later         |     |     |
                         [DC01][LNX01][MON01]
                          .10    .20    .30
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
    +-- MGMT     10.10.30.0/24   gateway .1
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
```

## Naming Rule

```text
ROLE + NUMBER

FW01
DC01
LNX01
MON01
CLIENT01
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

This file will be updated as the architecture changes and as new chapters introduce important concepts.
