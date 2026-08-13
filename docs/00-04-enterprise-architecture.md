# Chapter 0.4 — Enterprise Architecture Design

## Objective

Design the fictional company environment before deploying infrastructure. This chapter defines the company structure, core systems, network zones, naming conventions, VM sizing, and the first logical topology.

## Fictional Company

**Name:** BelkaCorp

**Size:** approximately 50 employees

**Departments:**

- Management
- Finance
- Human Resources
- Sales
- IT

The company is large enough to justify centralized identity, segmented networking, monitoring, backups, access control, and automation, while remaining small enough to reproduce on a single workstation.

## Initial Infrastructure Roles

| Name | Role | Planned OS / appliance | Initial purpose |
|---|---|---|---|
| FW01 | Firewall / router | OPNsense | Gateway, routing, segmentation, firewall policy |
| DC01 | Domain controller | Windows Server | Active Directory Domain Services and DNS |
| CLIENT01 | User workstation | Windows 11 | Domain-joined employee/client testing |
| LNX01 | Linux server | Ubuntu Server | Linux administration and later application/services work |
| MON01 | Monitoring server | Linux | Monitoring and observability later in the project |

Additional servers will only be added when a chapter creates a real need for them.

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

The Hyper-V Default Switch is used only as the simulated upstream/WAN side for `FW01`. The enterprise networks themselves remain separate from the host's normal home network.

## Naming Convention

Infrastructure devices use short uppercase functional names:

```text
<ROLE><NUMBER>
```

Examples:

- `FW01` — first firewall
- `DC01` — first domain controller
- `LNX01` — first general Linux server
- `MON01` — first monitoring server
- `CLIENT01` — first Windows client

The convention is intentionally simple and expandable.

## Network Zones and IPv4 Plan

BelkaCorp reserves `10.10.0.0/16` as the private lab address space. This `/16` is an address allocation/supernet for the lab; it is not itself a router or host. Individual functional networks are carved from it as `/24` subnets.

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

`FW01` owns the `.1` gateway address in each internal network.

### Planned User DHCP Range

The USERS network will later use DHCP for employee endpoints:

```text
10.10.10.100 - 10.10.10.199
```

`CLIENT01` will eventually obtain an address from this pool rather than receiving a manually assigned user IP.

## Hyper-V Virtual-Switch Layout

| Switch | Hyper-V type | Connected systems | Purpose |
|---|---|---|---|
| Default Switch | Hyper-V managed/NAT | `FW01` WAN NIC | Simulated upstream/Internet access |
| `vSW-USERS` | Private | `FW01`, `CLIENT01` | Employee endpoint network |
| `vSW-SERVERS` | Private | `FW01`, `DC01`, `LNX01`, `MON01` | Server network |
| `vSW-MGMT` | Internal | `FW01`, Windows host, later management systems | Lab administration path |

### Why these switch types?

- **Private** means only VMs attached to that switch communicate on it. This keeps USERS and SERVERS isolated from the physical Windows host unless `FW01` explicitly routes permitted traffic.
- **Internal** means VMs and the Windows host can communicate on that virtual switch. This is appropriate for the MGMT network because the physical desktop is also the administrator workstation.
- The host-side `vEthernet (vSW-MGMT)` adapter will later receive a static management address without a default gateway, so it does not replace or interfere with the desktop's normal Internet route.

## Inter-Subnet Routing Quick Note

Example:

```text
CLIENT01 = 10.10.10.125/24
DC01     = 10.10.20.10/24
```

`CLIENT01` compares the destination to its own `/24` network and determines that `10.10.20.10` is remote. It sends the packet toward its default gateway `10.10.10.1` (`FW01`). `FW01` checks its routing table, sees that `10.10.20.0/24` is directly connected through its SERVERS interface, and forwards the packet onto that network if firewall policy permits it.

The reserved `10.10.0.0/16` does not receive or forward the packet; it is simply the larger address block from which the lab subnets were allocated.

## Host Resource Budget

Physical host capacity:

- 12 cores / 24 logical processors
- 31.1 GB RAM
- ~954 GB SSD

Initial conservative VM targets:

| VM | vCPU | RAM target | Disk target |
|---|---:|---:|---:|
| FW01 | 2 | 2 GB | 16–20 GB |
| DC01 | 2 | 4 GB | 50–60 GB |
| CLIENT01 | 2–4 | 4–6 GB | 60–80 GB |
| LNX01 | 2 | 2–4 GB | 30–40 GB |
| MON01 | 2 | 2–4 GB | 30–50 GB |

Not all machines need to run simultaneously. VM sizing will be adjusted based on observed host usage rather than treating these numbers as fixed production requirements.

## Design Principles

1. Build only what has a clear purpose.
2. Separate user, server, and management traffic.
3. Keep the Windows host usable while the lab is running.
4. Prefer reproducible configuration and documentation.
5. Introduce automation only after the manual infrastructure is understood.
6. Preserve evidence of design decisions and troubleshooting.

## Chapter 0.4 Status

- [x] Define company and departments
- [x] Define initial infrastructure roles
- [x] Define naming convention
- [x] Define network zones
- [x] Define initial VM resource budget
- [x] Choose exact IPv4 subnets and gateway addresses
- [x] Define Hyper-V virtual-switch layout
- [x] Produce the finalized logical architecture schema
- [x] Record the design rationale

**Chapter 0.4 design is complete.** The next stage is Chapter 1: enable and validate Hyper-V, then reproduce this design as actual virtual infrastructure.
