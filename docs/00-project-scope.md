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
- [ ] Verify exact Windows build/version because the first PowerShell report may not identify Windows 11 correctly
- [ ] Verify why the CPU currently reports only 12 logical processors although the installed model supports SMT
- [ ] Identify the currently active Windows hypervisor / virtualization features
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
| Logical processors reported | 12 — requires verification |
| RAM | 31.1 GB usable/reportable |
| Storage | SPCC M.2 PCIe SSD, 953.87 GB |
| Architecture | 64-bit |
| Firmware virtualization | Enabled |
| Windows hypervisor detected | Yes |
| Windows product report | Windows 10 Pro, version value `2009` — exact build pending verification |

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

## Design Decisions

### DD-001 — Single-desktop lab

**Decision:** All required project work will run from one Windows desktop using virtualization and, where useful, network simulation/emulation.

**Reasoning:** This removes the operational friction of moving between physical machines while still allowing realistic isolated servers, clients, firewalls, and network topologies to be built and tested.

**Consequence:** Desktop RAM, CPU virtualization support, available storage, and Windows edition become important design constraints. Resource allocation will need to be managed carefully when several VMs are running simultaneously.

### DD-002 — Do not select the hypervisor from incomplete inventory data

**Decision:** Do not install or commit to VMware Workstation, Hyper-V, or another desktop hypervisor until the currently active Windows hypervisor state and exact Windows build have been verified.

**Reasoning:** The first audit reports `HypervisorPresent = True`, while VirtualBox host-only adapters also exist. Installing multiple virtualization stacks without understanding the current state can introduce compatibility/performance issues and would be poor infrastructure practice.

## Evidence

Only relevant sanitized output/screenshots will be stored in this repository.

Raw command output containing no useful portfolio evidence does not need to be committed simply for completeness.

## Completion Criteria

Chapter 0 is complete when:

1. The desktop hardware, Windows edition/build, and interfaces are documented.
2. Virtualization capability and the currently active hypervisor state are confirmed.
3. The virtualization platform and network-lab tooling have been selected with stated reasons.
4. An initial lab architecture has been designed.
5. The next implementation chapter can begin without guessing about the environment.
