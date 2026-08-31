# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** 3.7 — Add the Windows host's specific BelkaCorp routes
- **Current step:** 3.8 — Validate routed traffic and initial firewall behavior
- **Open issues:** SERVERS-to-MGMT ping currently fails; exact firewall decision still needs direct OPNsense live-log confirmation

## Verified Live State

`TEST01` is temporarily being used as a SERVERS-network validation endpoint. Its persistent Netplan configuration was not changed; only the live address and route were replaced for this test.

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
                                |                |
                             TEST01          Windows host
                         10.10.20.50/24      10.10.30.10/24
                         gw 10.10.20.1
                         [TEMPORARY]
```

Verified behavior:

- FW01 MGMT is `10.10.30.1/24` on `LAN / hn3` and is reachable from Windows.
- USERS is `10.10.10.1/24` on `OPT1 / hn1`.
- SERVERS is `10.10.20.1/24` on `OPT2 / hn2`.
- OPNsense has directly connected routes for all three BelkaCorp `/24` networks and a default route through WAN / `hn0`.
- Windows has persistent routes for USERS and SERVERS through FW01 while preserving its normal Wi-Fi default route.
- TEST01 is temporarily attached to `vSW-SERVERS` and has live address `10.10.20.50/24` with default gateway `10.10.20.1`.
- Windows `10.10.30.10` successfully pings TEST01 `10.10.20.50` with 0% loss.
- Windows traceroute to TEST01 shows hop 1 `10.10.30.1` and hop 2 `10.10.20.50`, proving real forwarding through FW01.
- A new ping initiated from TEST01 `10.10.20.50` to Windows `10.10.30.10` returns 100% loss. The exact blocking component is not yet formally attributed; the next check is the OPNsense live firewall log.

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

Connectivity validation from FW01 verified:

```text
FW01 -> 10.10.30.10   4/4 replies, 0% loss
FW01 -> 1.1.1.1       4/4 replies, 0% loss
```

## Verified Windows Routing State

```text
0.0.0.0/0       -> 192.168.0.1 -> Wi-Fi
10.10.10.0/24   -> 10.10.30.1  -> vEthernet (vSW-MGMT)
10.10.20.0/24   -> 10.10.30.1  -> vEthernet (vSW-MGMT)
```

The active routing table and `PersistentStore` both contain the two BelkaCorp `/24` routes. `Find-NetRoute` verified:

```text
10.10.10.50 -> vEthernet (vSW-MGMT), next hop 10.10.30.1
10.10.20.50 -> vEthernet (vSW-MGMT), next hop 10.10.30.1
1.1.1.1     -> Wi-Fi, next hop 192.168.0.1
```

## Step 3.8 Temporary Validation State

I moved TEST01 to `vSW-SERVERS` only for cross-subnet validation and used temporary live Linux commands rather than editing Netplan:

```text
TEST01 switch      vSW-SERVERS
TEST01 IPv4        10.10.20.50/24
TEST01 gateway     10.10.20.1
Persistent Netplan unchanged
```

Successful MGMT-to-SERVERS test:

```text
Windows 10.10.30.10 -> TEST01 10.10.20.50
ping                 4/4 replies, 0% loss
tracert hop 1        10.10.30.1
tracert hop 2        10.10.20.50
```

Observed reverse-direction failure:

```text
TEST01 10.10.20.50 -> Windows 10.10.30.10
4 transmitted, 0 received, 100% loss
```

This reverse failure is not yet documented as a confirmed firewall block. OPNsense live-log evidence is required before assigning the cause.

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
03-17a-mgmt-to-servers-routed-traffic.png           COMMITTED
03-17b-servers-to-mgmt-connectivity-blocked.png     COMMITTED
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

Open OPNsense's live firewall log, filter around the SERVERS/OPT2 traffic if useful, and repeat the TEST01 `10.10.20.50 -> 10.10.30.10` ping. Capture the firewall decision directly. Do not add or modify firewall rules before that observation is recorded.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read `docs/03-firewall-routing.md`.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so the current step remains accurate.
