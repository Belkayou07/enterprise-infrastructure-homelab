# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.6 — Verify management connectivity and routing foundation
- **Current step:** 3.7 — Add the Windows host's specific BelkaCorp routes
- **Open issues:** None currently blocking progress

## Verified Live State

```text
                         Hyper-V Default Switch
                                  |
                              WAN / hn0
                                 FW01
                            OPNsense 26.7
               +----------------+----------------+
               |                |                |
          USERS / hn1      SERVERS / hn2     MGMT / hn3
          10.10.10.1       10.10.20.1        10.10.30.1
               |                |                |
          vSW-USERS        vSW-SERVERS        vSW-MGMT
                                                |
                              +-----------------+-----------------+
                              |                                   |
                       Windows host                            TEST01
                       10.10.30.10/24                         10.10.30.20/24
```

Verified behavior:

- FW01 MGMT is configured as `10.10.30.1/24` on `LAN / hn3`.
- The Windows host at `10.10.30.10/24` successfully reaches `10.10.30.1`, and the OPNsense HTTPS Web GUI is reachable at `https://10.10.30.1`.
- USERS is configured as `10.10.10.1/24` on `OPT1 / hn1`.
- SERVERS is configured as `10.10.20.1/24` on `OPT2 / hn2`.
- OPNsense has directly connected routes for all three BelkaCorp `/24` networks on the expected interfaces.
- OPNsense has a default route through WAN / `hn0`.
- FW01 successfully pings the Windows MGMT host `10.10.30.10` with 0% loss.
- FW01 successfully pings public IP `1.1.1.1` with 0% loss, proving WAN IP connectivity at this checkpoint.

## Chapter 3 Deployment State

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

The OPNsense 26.7 installation is complete. The first post-install start fell through to PXE, but the VHDX and firmware boot chain were investigated and corrected by explicitly prioritizing `FW01.vhdx`. The incident is closed in `troubleshooting/006-fw01-post-install-pxe-boot.md`.

### Verified interface identity and roles

```text
Hyper-V role   OPNsense NIC   OPNsense role   IPv4
-----------------------------------------------------------
WAN            hn0            WAN             DHCP
USERS          hn1            OPT1            10.10.10.1/24
SERVERS        hn2            OPT2            10.10.20.1/24
MGMT           hn3            LAN             10.10.30.1/24
```

MAC-address verification established the interface identity before these roles were assigned.

### Verified FW01 routing foundation

The OPNsense IPv4 routing table contains:

```text
10.10.10.0/24   directly connected   hn1   USERS
10.10.20.0/24   directly connected   hn2   SERVERS
10.10.30.0/24   directly connected   hn3   MGMT
default         upstream gateway     hn0   WAN
```

At the validation checkpoint, the WAN side was on a Hyper-V Default Switch DHCP network and the routing table showed the default gateway through `hn0`. Because this WAN addressing is supplied dynamically, the exact WAN lease may change.

Explicit route lookups verified:

```text
10.10.10.50 -> hn1
10.10.20.50 -> hn2
10.10.30.10 -> hn3
```

The `.50` destinations do not need to exist; these lookups test route selection only.

Connectivity validation verified:

```text
FW01 -> 10.10.30.10   4/4 replies, 0% loss
FW01 -> 1.1.1.1       4/4 replies, 0% loss
```

This completes Step 3.6. Routing is present at Layer 3; firewall policy remains a separate control that will be validated later.

## Completed Network Design

BelkaCorp allocation block:

```text
10.10.0.0/16
```

Current subnets:

```text
USERS    10.10.10.0/24   gateway 10.10.10.1 [CONFIGURED + ROUTE VERIFIED]
SERVERS  10.10.20.0/24   gateway 10.10.20.1 [CONFIGURED + ROUTE VERIFIED]
MGMT     10.10.30.0/24   gateway 10.10.30.1 [CONFIGURED + VERIFIED]
```

The Windows host keeps its normal Wi-Fi default route. It still needs specific routes for USERS and SERVERS through the reachable MGMT next hop `10.10.30.1`.

## Evidence Status

The current Chapter 3 evidence is committed and verified under `screenshots/chapter-03/`:

```text
03-01-fw01-predeployment-baseline.png              COMMITTED
03-02-opnsense-download-hash-verification.png      COMMITTED
03-03-opnsense-iso-staged.png                      COMMITTED
03-04-fw01-wizard-summary.png                      COMMITTED
03-05-fw01-initial-preboot-audit.png                COMMITTED
03-06-fw01-final-preboot-verification.png           COMMITTED
03-07-opnsense-boot-menu.png                        COMMITTED
03-08-fw01-pxe-after-install.png                    COMMITTED
03-09-opnsense-installed-first-disk-boot.png        COMMITTED
03-10-fw01-hyperv-nic-mac-map.png                   COMMITTED
03-11-opnsense-guest-nic-mac-map.png                COMMITTED
03-12-opnsense-interface-role-assignment.png        COMMITTED
03-13a-opnsense-mgmt-ip-configured.png              COMMITTED
03-13b-mgmt-connectivity-and-webgui.png              COMMITTED
03-14-opnsense-internal-gateways-configured.png     COMMITTED
03-15a-opnsense-routing-table.png                   COMMITTED
03-15b-opnsense-route-lookups.png                   COMMITTED
03-15c-opnsense-connectivity-validation.png         COMMITTED
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

Continue **3.7 — Add the Windows host's specific BelkaCorp routes**.

The Windows host must preserve its normal Internet default route on Wi-Fi while adding only these lab-specific destinations through FW01:

```text
10.10.10.0/24 -> next hop 10.10.30.1 via vEthernet (vSW-MGMT)
10.10.20.0/24 -> next hop 10.10.30.1 via vEthernet (vSW-MGMT)
```

After adding them, verify route selection and persistence before moving into cross-subnet traffic and firewall-policy validation.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.