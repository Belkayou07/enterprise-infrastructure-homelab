# Current Project State

> Operational handoff checkpoint for session continuity. This file is not a technical deliverable; it exists so future work can resume from the correct verified state without reconstructing the entire chat history.

## Current Position

- **Current chapter:** Chapter 2 — Network Architecture and IP Design
- **Last completed step:** 2.2 — Subnet and Address Allocation
- **Next step:** 2.3 — Define Layer-2 and Layer-3 Boundaries
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

Continue with **2.3 — Layer-2 / Layer-3 Boundaries**.

The goal is to define clearly:

- what each Hyper-V virtual switch does at Layer 2;
- which systems share the same broadcast domain;
- where Layer 2 ends;
- where routing begins;
- which Layer-3 responsibilities will later belong to `FW01`.

Do **not** deploy `FW01` yet unless the Chapter 2 design explicitly reaches the implementation point.

## Resume Rules

When resuming after a chat/session break:

1. Read this file first.
2. Read the current chapter document (`docs/02-network-architecture.md`).
3. Check the latest `main` commits if the two disagree.
4. Use actual committed evidence to confirm completed implementation/verification work.
5. Do not redo completed steps simply because conversational context is missing.
6. Update this file after each meaningful project milestone so `Last completed step` and `Next step` remain accurate.
.
