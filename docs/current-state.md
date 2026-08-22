# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 2 — Network Architecture and IP Design
- **Last completed step:** 2.5 — Define DHCP and DNS Ownership
- **Next step:** 2.6 — Produce the Final Network Implementation Plan
- **Open issues:** None currently blocking progress

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
- A route lookup toward `10.10.20.20` is currently unreachable, which is expected because no BelkaCorp router exists yet.
- `FW01` has **not** been deployed yet.

## Current Network Design

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

Planned addressing:

```text
USERS
10.10.10.1          FW01
10.10.10.100-.199   planned DHCP range

SERVERS
10.10.20.1          FW01
10.10.20.10         DC01
10.10.20.20         LNX01
10.10.20.30         MON01

MGMT
10.10.30.1          FW01
10.10.30.10         Windows Hyper-V host
10.10.30.20         TEST01 temporary validation VM
```

Addressing convention:

```text
.1           FW01 / default gateway
.2 - .9      reserved infrastructure
.10 - .49    static infrastructure / servers
.50 - .99    future static systems
.100 - .199  dynamic clients where DHCP is appropriate
.200 - .254  spare / future use
```

## Layer Boundaries

Each custom Hyper-V switch is a distinct Layer-2 segment and broadcast domain:

```text
vSW-USERS    -> USERS   -> 10.10.10.0/24
vSW-SERVERS  -> SERVERS -> 10.10.20.0/24
vSW-MGMT     -> MGMT    -> 10.10.30.0/24
```

Same-subnet communication is direct at Layer 2 through ARP and MAC-address forwarding. Cross-subnet communication requires Layer-3 routing. `FW01` will later connect to each segment with a separate virtual NIC and will route between them rather than bridge them into one Layer-2 network.

For cross-subnet traffic, the IP destination remains the final remote host while the Ethernet destination MAC identifies the next local Layer-2 hop, initially the local `FW01` gateway interface.

## Gateway and Routing Design

Normal BelkaCorp hosts will use the `FW01` address in their own local subnet as default gateway:

```text
USERS    -> 10.10.10.1
SERVERS  -> 10.10.20.1
MGMT     -> 10.10.30.1 where appropriate
```

Once those interfaces exist, `FW01` will automatically have directly connected routes to `10.10.10.0/24`, `10.10.20.0/24`, and `10.10.30.0/24`; no manual static routes are required on `FW01` for those three networks.

The Windows host will keep its normal Wi-Fi default route through `192.168.0.1`. After `FW01` exists, the design is to use specific routes for `10.10.10.0/24` and `10.10.20.0/24` through `10.10.30.1`, allowing longest-prefix matching to send BelkaCorp traffic to `FW01` without moving normal Internet traffic away from Wi-Fi.

Routing must work in both directions. Remote hosts return traffic through their own local `FW01` gateway interface, not through the `.1` address of the destination subnet.

## DHCP and DNS Design

`DC01` will own both DHCP and internal DNS for the Windows domain environment.

Planned USERS DHCP scope:

```text
Network        10.10.10.0/24
Dynamic pool   10.10.10.100 - 10.10.10.199
Gateway        10.10.10.1
DNS server     10.10.20.10
```

`FW01` will relay DHCP requests from the USERS broadcast domain toward `DC01` at `10.10.20.10`. `FW01` does not own the DHCP lease pool; `DC01` does.

Servers and management systems remain primarily statically addressed.

Domain clients will use `DC01` at `10.10.20.10` for DNS. This allows Active Directory service discovery through internal DNS records for domain controllers and services such as Kerberos and LDAP. Public Internet names can be resolved through upstream DNS forwarding or recursion configured on `DC01` later.

## Hyper-V Foundation

Chapter 1 is complete. The current verified foundation includes:

- Hyper-V enabled and validated.
- BelkaCorp VM/VHDX storage paths configured under `C:\Hyper-V\BelkaCorp`.
- `vSW-MGMT` = Internal.
- `vSW-SERVERS` = Private.
- `vSW-USERS` = Private.
- TEST01 runs Ubuntu Server as a Generation 2 VM.
- TEST01 Dynamic Memory is intentionally configured at 2048 MB startup / 1536 MB minimum / 4096 MB maximum.
- TEST01 automatic checkpoints are disabled.
- TEST01 currently runs directly from `TEST01.vhdx` with no active `.avhdx` checkpoint chain.

## Immediate Next Work

Continue with **2.6 — Final Network Implementation Plan**.

The goal is to turn the completed Chapter 2 design into an implementation-ready sequence for the next chapters, including:

- `FW01` virtual NIC placement and planned addresses;
- the order in which routing should be introduced and verified;
- Windows-host BelkaCorp route changes only after the MGMT gateway exists;
- later deployment of `DC01` with static addressing, DNS, and DHCP;
- DHCP relay dependency between USERS and `DC01`;
- verification points that must succeed before moving forward.

Do **not** deploy `FW01`, `DC01`, DHCP, DNS, or Active Directory as part of this planning step. The actual deployment belongs to the following implementation chapters.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read the current chapter document (`docs/02-network-architecture.md`).
3. Check the latest `main` commits if the two disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so `Last completed step` and `Next step` remain accurate.
