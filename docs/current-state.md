# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 3 — Firewall and Routing
- **Last completed step:** Chapter 2.7 — Final Chapter 2 Audit
- **Next step:** Begin `FW01` deployment and pre-boot verification
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

## Layer and Routing Model

Each custom Hyper-V switch is a distinct Layer-2 segment:

```text
vSW-USERS    -> USERS   -> 10.10.10.0/24
vSW-SERVERS  -> SERVERS -> 10.10.20.0/24
vSW-MGMT     -> MGMT    -> 10.10.30.0/24
```

`FW01` will connect to all three internal segments and route between them. Normal hosts use the `FW01` address on their own subnet as gateway. The Windows host will keep its Wi-Fi default route and will receive specific BelkaCorp routes only after `FW01` exists and `10.10.30.1` is reachable.

## DHCP and DNS Ownership

```text
FW01
- routing
- firewall policy
- DHCP relay between routed subnets

DC01
- Active Directory Domain Services later
- internal / AD-integrated DNS
- Windows Server DHCP

CLIENT01
- DHCP client
- uses 10.10.20.10 for DNS
```

The USERS DHCP scope is planned as `10.10.10.100-10.10.10.199`, with gateway `10.10.10.1` and DNS server `10.10.20.10`.

## Chapter 3 Starting Point

The first implementation dependency is `FW01`.

Planned interfaces:

```text
FW01 / OPNsense
|
+-- NIC 1  WAN      -> Hyper-V Default Switch
+-- NIC 2  USERS    -> vSW-USERS    -> 10.10.10.1/24
+-- NIC 3  SERVERS  -> vSW-SERVERS  -> 10.10.20.1/24
+-- NIC 4  MGMT     -> vSW-MGMT     -> 10.10.30.1/24
```

The first useful validation after installation will be the MGMT interface because the Windows host and TEST01 already provide known-good endpoints on `10.10.30.0/24`.

Do **not** add the Windows routes to `10.10.10.0/24` and `10.10.20.0/24` until `FW01` is actually configured and `10.10.30.1` is reachable.

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

Start **Chapter 3 — Firewall and Routing** by preparing and deploying `FW01` as an OPNsense VM. Work one verified step at a time: define the VM configuration, verify the four virtual NIC attachments before installation, then install/configure OPNsense without jumping ahead to Windows routes or later services.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read the most relevant current chapter document.
3. Check the latest `main` commits if files disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so `Last completed step` and `Next step` remain accurate.
