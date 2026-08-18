# Chapter 0 — Enterprise Architecture Design

## Goal

I designed the fictional company environment before deploying infrastructure. In this design document I defined the company structure, core systems, network zones, naming conventions, VM sizing, and the initial logical topology that later chapters will implement.

## Fictional Company

**Name:** BelkaCorp  
**Size:** approximately 50 employees

**Departments:**

- Management
- Finance
- Human Resources
- Sales
- IT

I chose this size because it is large enough to justify centralized identity, segmented networking, monitoring, backups, access control, and automation while remaining small enough to reproduce on one physical workstation.

## Initial Infrastructure Roles

| Name | Role | Planned OS / appliance | Initial purpose |
|---|---|---|---|
| FW01 | Firewall / router | OPNsense | Gateway, routing, segmentation, firewall policy |
| DC01 | Domain controller | Windows Server | Active Directory Domain Services and DNS |
| CLIENT01 | User workstation | Windows 11 | Domain-joined employee/client testing |
| LNX01 | Linux server | Ubuntu Server | Linux administration and later application/services work |
| MON01 | Monitoring server | Linux | Monitoring and observability later in the project |

I will only add more servers when a later chapter creates a real requirement for them.

## Final Logical Architecture

```text
                        INTERNET
                           |
                 Windows host connectivity
                           |
                 Hyper-V Default Switch
                           |
                      WAN NIC
                        [FW01]
                     OPNsense
        +------------------+------------------+
        |                  |                  |
    USERS NIC          SERVERS NIC         MGMT NIC
  10.10.10.1         10.10.20.1         10.10.30.1
        |                  |                  |
   vSW-USERS          vSW-SERVERS          vSW-MGMT
    Private              Private             Internal
        |                  |                  |
   [CLIENT01]        +-----+-----+       Windows host
    DHCP later       |     |     |       admin access
                   [DC01][LNX01][MON01]
                    .10    .20    .30
```

I use the Hyper-V Default Switch only as the simulated upstream/WAN side for `FW01`. The BelkaCorp enterprise networks remain separate from my normal home network.

## Naming Convention

I use short uppercase functional names for infrastructure devices:

```text
<ROLE><NUMBER>
```

Examples:

- `FW01` — first firewall
- `DC01` — first domain controller
- `LNX01` — first general Linux server
- `MON01` — first monitoring server
- `CLIENT01` — first Windows client

I intentionally kept the convention simple and expandable.

## Network Zones and IPv4 Plan

I reserved `10.10.0.0/16` as the BelkaCorp private lab address space. I use this `/16` as an address allocation/supernet for the lab; it is not itself a router or host. I allocate individual functional networks from it as `/24` subnets.

| Zone | Subnet | Default gateway | Purpose |
|---|---|---|---|
| USERS | `10.10.10.0/24` | `10.10.10.1` | Employee workstations |
| SERVERS | `10.10.20.0/24` | `10.10.20.1` | Internal servers and infrastructure services |
| MGMT | `10.10.30.0/24` | `10.10.30.1` | Administrative and management access |

### Initial Static Server Addresses

| Host | Address | Network |
|---|---|---|
| DC01 | `10.10.20.10/24` | SERVERS |
| LNX01 | `10.10.20.20/24` | SERVERS |
| MON01 | `10.10.20.30/24` | SERVERS |

I designed `FW01` to own the `.1` gateway address in each internal network.

### Planned User DHCP Range

I planned the USERS network to use DHCP for employee endpoints:

```text
10.10.10.100 - 10.10.10.199
```

`CLIENT01` will eventually obtain an address from this pool instead of receiving a manually assigned user IP.

## Hyper-V Virtual-Switch Layout

| Switch | Hyper-V type | Connected systems | Purpose |
|---|---|---|---|
| Default Switch | Hyper-V managed/NAT | `FW01` WAN NIC | Simulated upstream/Internet access |
| `vSW-USERS` | Private | `FW01`, `CLIENT01` | Employee endpoint network |
| `vSW-SERVERS` | Private | `FW01`, `DC01`, `LNX01`, `MON01` | Server network |
| `vSW-MGMT` | Internal | `FW01`, Windows host, later management systems | Lab administration path |

### Why I chose these switch types

- I use **Private** switches for USERS and SERVERS so only VMs attached to those switches communicate directly on them. The Windows host cannot bypass `FW01` and join those networks directly.
- I use an **Internal** switch for MGMT so my Windows desktop can act as the administrator workstation and communicate with VMs on the management network.
- I planned the host-side `vEthernet (vSW-MGMT)` adapter to use a static management address without a default gateway so the lab does not replace or interfere with my normal Windows Internet route.

## Inter-Subnet Routing Quick Note

Example:

```text
CLIENT01 = 10.10.10.125/24
DC01     = 10.10.20.10/24
```

`CLIENT01` compares the destination to its own `/24` network and determines that `10.10.20.10` is remote. It sends the packet to its default gateway `10.10.10.1` on `FW01`. `FW01` checks its routing table, sees that `10.10.20.0/24` is directly connected through the SERVERS interface, and forwards the packet onto that network if firewall policy permits it.

The reserved `10.10.0.0/16` does not receive or forward the packet; it is simply the larger address block from which I allocated the lab subnets.

## Host Resource Budget

My physical host capacity at design time was:

- 12 cores / 24 logical processors
- 31.1 GB RAM
- ~954 GB SSD

I chose these initial conservative VM targets:

| VM | vCPU | RAM target | Disk target |
|---|---:|---:|---:|
| FW01 | 2 | 2 GB | 16–20 GB |
| DC01 | 2 | 4 GB | 50–60 GB |
| CLIENT01 | 2–4 | 4–6 GB | 60–80 GB |
| LNX01 | 2 | 2–4 GB | 30–40 GB |
| MON01 | 2 | 2–4 GB RAM | 30–50 GB |

I do not expect every VM to run simultaneously. I will adjust sizing from observed host usage rather than treating these numbers as fixed production requirements.

## Design Principles

1. I build only what has a clear purpose.
2. I separate user, server, and management traffic.
3. I keep the Windows host usable while the lab is running.
4. I prefer reproducible configuration and documentation.
5. I introduce automation only after I understand the manual infrastructure.
6. I preserve evidence of design decisions and troubleshooting.

## Architecture Design Status

- [x] Define company and departments
- [x] Define initial infrastructure roles
- [x] Define naming convention
- [x] Define network zones
- [x] Define initial VM resource budget
- [x] Choose exact IPv4 subnets and gateway addresses
- [x] Define Hyper-V virtual-switch layout
- [x] Produce the finalized logical architecture schema
- [x] Record the design rationale

**Status: COMPLETE.** I finished the initial enterprise design before moving into Chapter 1 to implement and validate the Hyper-V platform.