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

## Quick Architecture Schema

```text
                          INTERNET
                              |
                              |
                           [ FW01 ]
                       OPNsense Firewall
                              |
              +---------------+---------------+
              |               |               |
          USERS NET       SERVERS NET      MGMT NET
              |               |               |
          [CLIENT01]      +----+----+       Admin
                          |    |    |       access
                        [DC01][LNX01][MON01]
```

This is the logical target. Hyper-V virtual switches and detailed subnetting will be designed before deployment.

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

## Network Zones — Draft

The lab will use separate logical zones rather than placing every machine in one flat network.

| Zone | Purpose |
|---|---|
| WAN | External/upstream connectivity |
| USERS | Employee workstations |
| SERVERS | Internal infrastructure and application servers |
| MGMT | Administrative/management access |

Exact subnets are not considered final until the IP addressing exercise is completed.

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

## Remaining Chapter 0.4 Tasks

- [x] Define company and departments
- [x] Define initial infrastructure roles
- [x] Define naming convention
- [x] Define draft network zones
- [x] Define initial VM resource budget
- [ ] Choose exact IPv4 subnets and gateway addresses
- [ ] Define Hyper-V virtual-switch layout
- [ ] Produce the finalized logical architecture schema
- [ ] Record the final design decision in Chapter 0 documentation
