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
Physical Windows Desktop
│
├── Administrator workstation / browser / terminal / Git
│
├── Virtualization platform
│   ├── Windows Server VM(s)
│   ├── Windows client VM(s)
│   ├── Linux server VM(s)
│   ├── Firewall/router VM
│   └── Monitoring/utility VM(s)
│
└── Network lab tooling
    └── Virtual/emulated routers, switches and isolated networks as required
```

The exact virtualization and network-emulation products will be selected after checking the desktop's CPU, RAM, storage, Windows edition, and virtualization capabilities.

## Chapter 0 Tasks

- [x] Record desktop hardware and reported Windows edition
- [x] Record available network interfaces
- [x] Verify exact Windows build/version
- [ ] Verify why the CPU currently exposes only 12 logical processors although the installed model supports 24 threads
- [ ] Identify which Windows virtualization optional features are enabled
- [x] Confirm that a Windows hypervisor is currently active
- [x] Confirm hardware virtualization support is enabled in firmware
- [x] Define the physical-host strategy: single desktop
- [ ] Choose the virtualization platform
- [ ] Choose network simulation/emulation tooling
- [ ] Define the first architecture diagram
- [ ] Define naming conventions
- [ ] Define an initial IP addressing plan
- [ ] Record Chapter 0 engineering decisions

## Hardware Inventory

### Desktop

Initial PowerShell audit performed on 2026-08-12.

| Component | Observed value |
|---|---|
| Manufacturer | Micro-Star International Co., Ltd. |
| Motherboard/system model | MS-7E26 |
| CPU | AMD Ryzen 9 7900 12-Core Processor |
| Physical CPU cores reported | 12 |
| Hardware thread count reported | 24 |
| Logical processors exposed to Windows | 12 — still under investigation |
| RAM | 31.1 GB usable/reportable |
| Storage | SPCC M.2 PCIe SSD, 953.87 GB |
| Architecture | 64-bit |
| Firmware virtualization | Enabled |
| Windows hypervisor detected | Yes |
| Operating system | Windows 11 Pro, version 25H2 |
| OS build | 26200.8973 |

Microsoft's official build information identifies build 26200.8973 as Windows 11 version 25H2. The registry's `ProductName` field still returned `Windows 10 Pro`; this is treated as legacy/compatibility metadata rather than the authoritative version indicator.

AMD specifies the Ryzen 9 7900 as a 12-core / 24-thread processor with SMT support. Windows currently reports only 12 logical processors while WMI reports `ThreadCount = 24`; this discrepancy will be investigated before VM CPU allocations are finalized.

The processor and memory capacity are sufficient for a multi-VM infrastructure lab, but resource allocation will be planned so the Windows host remains responsive.

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

The existing VirtualBox adapters show that virtualization networking has already been installed on the host. This will be considered before selecting or installing another hypervisor.

## Virtualization State

`systeminfo` reports:

```text
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

The same system report includes Hypervisor-Enforced Code Integrity (HVCI). Windows virtualization-based security and Memory Integrity use the Windows hypervisor, so the presence of an active hypervisor does **not by itself prove** that the full Hyper-V VM-management feature is installed.

The optional Windows features still need to be queried from an elevated PowerShell session before the lab hypervisor is selected.

## Design Decisions

### DD-001 — Single-desktop lab

**Decision:** All required project work will run from one Windows desktop using virtualization and, where useful, network simulation/emulation.

**Reasoning:** This removes the operational friction of moving between physical machines while still allowing realistic isolated servers, clients, firewalls, and network topologies to be built and tested.

**Consequence:** Desktop RAM, CPU virtualization support, available storage, and Windows edition become important design constraints. Resource allocation will need to be managed carefully when several VMs are running simultaneously.

### DD-002 — Do not select the hypervisor from incomplete inventory data

**Decision:** Do not install or commit to VMware Workstation, Hyper-V, or another desktop hypervisor until the currently active Windows virtualization features have been verified.

**Reasoning:** The host already runs the Windows hypervisor for at least virtualization-based security, while VirtualBox host-only adapters also exist. Installing or enabling additional virtualization components without understanding the current state can create unnecessary compatibility/performance problems.

## Evidence

Only relevant sanitized output/screenshots will be stored in this repository.

Raw command output containing no useful portfolio evidence does not need to be committed simply for completeness.

## Completion Criteria

Chapter 0 is complete when:

1. The desktop hardware, Windows edition/build, and interfaces are documented.
2. Virtualization capability and the currently active Windows virtualization features are confirmed.
3. The CPU logical-processor discrepancy is understood or explicitly accepted as a host constraint.
4. The virtualization platform and network-lab tooling have been selected with stated reasons.
5. An initial lab architecture has been designed.
6. The next implementation chapter can begin without guessing about the environment.
