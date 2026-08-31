# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.7 — Add the Windows host's specific BelkaCorp routes
- **Current step:** 3.8 — Validate routed traffic and initial firewall behavior
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

- FW01 MGMT is `10.10.30.1/24` on `LAN / hn3` and is reachable from the Windows host.
- USERS is `10.10.10.1/24` on `OPT1 / hn1`.
- SERVERS is `10.10.20.1/24` on `OPT2 / hn2`.
- OPNsense has directly connected routes for all three BelkaCorp `/24` networks and a default route through WAN / `hn0`.
- FW01 successfully reaches the Windows MGMT host and public IP `1.1.1.1`.
- Windows now has persistent routes for USERS and SERVERS through FW01 while preserving its normal Wi-Fi default route.

## Verified FW01 Platform State

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

The OPNsense 26.7 installation is complete. The post-install PXE incident was resolved by explicitly prioritizing `FW01.vhdx` in Generation 2 firmware and is documented in `troubleshooting/006-fw01-post-install-pxe-boot.md`.

## Verified Interface Identity and Roles

```text
Hyper-V role   OPNsense NIC   OPNsense role   IPv4
-----------------------------------------------------------
WAN            hn0            WAN             DHCP
USERS          hn1            OPT1            10.10.10.1/24
SERVERS        hn2            OPT2            10.10.20.1/24
MGMT           hn3            LAN             10.10.30.1/24
```

MAC-address verification established the interface identity before these roles were assigned.

## Verified FW01 Routing Foundation

```text
10.10.10.0/24   directly connected   hn1   USERS
10.10.20.0/24   directly connected   hn2   SERVERS
10.10.30.0/24   directly connected   hn3   MGMT
default         upstream gateway     hn0   WAN
```

Explicit route lookups verified:

```text
10.10.10.50 -> hn1
10.10.20.50 -> hn2
10.10.30.10 -> hn3
```

Connectivity validation verified:

```text
FW01 -> 10.10.30.10   4/4 replies, 0% loss
FW01 -> 1.1.1.1       4/4 replies, 0% loss
```

## Verified Windows Routing State

The Windows MGMT adapter is connected as `vEthernet (vSW-MGMT)` with DHCP disabled. The host's normal Internet default route remains:

```text
0.0.0.0/0 -> 192.168.0.1 -> Wi-Fi
```

The following persistent BelkaCorp routes are now configured:

```text
10.10.10.0/24 -> 10.10.30.1 -> vEthernet (vSW-MGMT)
10.10.20.0/24 -> 10.10.30.1 -> vEthernet (vSW-MGMT)
```

The active routing table and `PersistentStore` both contain the two `/24` routes. `Find-NetRoute` verified actual route selection:

```text
10.10.10.50 -> vEthernet (vSW-MGMT), next hop 10.10.30.1
10.10.20.50 -> vEthernet (vSW-MGMT), next hop 10.10.30.1
1.1.1.1     -> Wi-Fi, next hop 192.168.0.1
```

This means Windows sends only USERS/SERVERS traffic through FW01 while ordinary Internet traffic continues through Wi-Fi.

## Completed Network Design

```text
USERS    10.10.10.0/24   gateway 10.10.10.1 [CONFIGURED + ROUTE VERIFIED]
SERVERS  10.10.20.0/24   gateway 10.10.20.1 [CONFIGURED + ROUTE VERIFIED]
MGMT     10.10.30.0/24   gateway 10.10.30.1 [CONFIGURED + VERIFIED]
```

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
03-16a-windows-belka-routes-configured.png          COMMITTED
03-16b-windows-route-selection.png                  COMMITTED
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

Continue **3.8 — Validate routed traffic and initial firewall behavior**.

First verify the routed Windows path to FW01's USERS and SERVERS interface addresses. Then use a real endpoint on another BelkaCorp subnet to test actual forwarded traffic and observe OPNsense firewall behavior. Do not treat route availability as proof that the firewall permits forwarding.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.
