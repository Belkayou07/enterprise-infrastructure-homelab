# Chapter 0 — Project Scope, Hardware Audit, and Repository Setup

## Objective

Establish the real constraints of the homelab before choosing the architecture. This chapter records the available desktop hardware, Windows environment, network access, and the design decisions that follow from those facts.

## Business Scenario

The lab will simulate the infrastructure of a small organization with multiple user groups, centralized services, segmented networking, security controls, monitoring, backup, and later automation/cloud integration.

The environment should be realistic enough to demonstrate junior system administration, network engineering, cloud, and DevOps skills while remaining practical on one physical workstation.

## Core Lab Constraint

The entire lab will be operated from a **single Windows desktop**.

The desktop will serve two roles:

1. **Physical administrator workstation** — the normal Windows environment used to manage, test, document, and troubleshoot the lab.
2. **Lab compute host** — virtual machines and network simulation/emulation tools will provide the servers, clients, firewalls, routers, switches, and other infrastructure required by later chapters.

The separate laptop is intentionally excluded from the required architecture so the project remains convenient to reproduce and operate from one machine. It may only be used later as an optional extension if there is a clear engineering reason.

## Planned Logical Model

```text
Physical Windows 11 Pro Desktop
│
├── Administrator workstation / browser / terminal / Git
│
├── Hyper-V
│   ├── Windows Server VM(s)
│   ├── Windows client VM(s)
│   ├── Linux server VM(s)
│   ├── Firewall/router VM
│   └── Monitoring/utility VM(s)
│
└── Network lab tooling
    └── GNS3 / GNS3 VM on Hyper-V as required
```

## Chapter 0 Tasks

- [x] Record desktop hardware and reported Windows edition
- [x] Record available network interfaces
- [x] Verify exact Windows build/version
- [ ] Verify why the CPU currently exposes only 12 logical processors although the installed model supports 24 threads
- [x] Identify the currently active Windows virtualization state
- [x] Confirm hardware virtualization support is enabled in firmware
- [x] Define the physical-host strategy: single desktop
- [x] Choose the virtualization platform: Hyper-V
- [x] Choose planned network simulation/emulation tooling: GNS3 with Hyper-V support
- [ ] Define the first architecture diagram
- [ ] Define naming conventions
- [ ] Define an initial IP addressing plan
- [ ] Record final Chapter 0 engineering decisions

## Hardware Inventory

### Desktop

Initial PowerShell audit performed on 2026-08-12.

| Component | Observed value |
|---|---|
| Manufacturer | Micro-Star International Co., Ltd. |
| Motherboard/system model | MS-7E26 |
| CPU | AMD Ryzen 9 7900 12-Core Processor |
| Physical CPU cores reported | 12 |
| Logical processors exposed to Windows | 12 — still under investigation |
| CPU hardware thread count | 24 |
| RAM | 31.1 GB usable/reportable |
| Storage | SPCC M.2 PCIe SSD, 953.87 GB |
| Architecture | 64-bit |
| Firmware virtualization | Enabled |
| Windows edition | Windows 11 Pro |
| Windows release/build | 25H2, build 26200.8973 |

The Ryzen 9 7900 supports 12 cores and 24 threads. Windows currently exposes only 12 logical processors, while WMI reports `ThreadCount = 24`. No `numproc` boot limit was found. The remaining cause will be checked before changing firmware settings.

The machine is already sufficient for the planned multi-VM lab even at the currently exposed 12 logical processors, provided VM resources are allocated conservatively.

### Additional Physical Hardware

Not required for the core project.

## Network Environment

Observed adapters during the initial audit:

| Adapter | State | Observed link speed / purpose |
|---|---|---|
| RZ616 Wi-Fi 6E 160MHz | Up | 144.4 Mbps; current host connectivity |
| Realtek Gaming 2.5GbE | Disconnected | Physical 2.5 GbE Ethernet adapter |
| Tailscale Tunnel | Up | Overlay/VPN adapter |
| VirtualBox Host-Only Ethernet Adapter | Up | 1 Gbps virtual host-only adapter |
| Additional VirtualBox Host-Only adapter | Not present | Stale/unused virtual adapter |

The VirtualBox adapters indicate a previous virtualization installation. They are not part of the planned final architecture and can be reviewed later if cleanup becomes useful.

## Current Windows Virtualization State

Verified from an elevated PowerShell session on 2026-08-12:

| Feature / setting | State |
|---|---|
| Microsoft-Hyper-V-All | Disabled |
| VirtualMachinePlatform | Enabled |
| HypervisorPlatform | Disabled |
| `hypervisorlaunchtype` | Auto |
| Virtualization-Based Security | Enabled and running |
| Memory Integrity / HVCI | Configured and running |

The Windows hypervisor is therefore already active for host security features even though the full Hyper-V VM-management feature has not yet been enabled.

## Design Decisions

### DD-001 — Single-desktop lab

**Decision:** All required project work will run from one Windows desktop using virtualization and, where useful, network simulation/emulation.

**Reasoning:** This removes the operational friction of moving between physical machines while still allowing realistic isolated servers, clients, firewalls, and network topologies to be built and tested.

### DD-002 — Preserve the current Windows security baseline

**Decision:** Keep the host's current VBS/Memory Integrity security baseline while building the lab.

**Reasoning:** The lab should demonstrate sensible infrastructure practice on the everyday workstation rather than weakening the host merely to accommodate a different virtualization stack.

### DD-003 — Hyper-V as the primary desktop hypervisor

**Decision:** Use Microsoft Hyper-V as the primary virtualization platform.

**Reasoning:** The host runs Windows 11 Pro, virtualization is enabled, the Microsoft hypervisor is already active for Windows security features, and Hyper-V integrates naturally with Windows administration and PowerShell. Current GNS3 releases also provide a Hyper-V-compatible GNS3 VM.

**Consequence:** Chapter 1 will enable and validate the full Hyper-V feature, establish VM storage conventions, create virtual networking, and deploy a small test VM before the enterprise services are built.

### DD-004 — GNS3 for advanced network emulation

**Decision:** Add GNS3 later when router/switch appliance emulation and larger network topologies become useful.

**Reasoning:** Hyper-V will host the main server/client infrastructure; GNS3 adds dedicated network-engineering capabilities without requiring a second physical machine.

## Evidence

Only relevant sanitized output/screenshots will be stored in this repository.

Raw command output containing no useful portfolio evidence does not need to be committed simply for completeness.

## Completion Criteria

Chapter 0 is complete when:

1. The desktop hardware, Windows edition/build, and interfaces are documented.
2. Virtualization capability and the current Windows virtualization state are confirmed.
3. The virtualization platform and network-lab tooling have been selected with stated reasons.
4. The CPU logical-processor anomaly has either been resolved or explicitly accepted as a known host constraint.
5. An initial lab architecture, naming convention, and IP plan have been designed.
6. The next implementation chapter can begin without guessing about the environment.
