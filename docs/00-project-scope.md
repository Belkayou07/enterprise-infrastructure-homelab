# Chapter 0 — Project Scope, Hardware Audit, and Repository Setup

## Goal

I established the real constraints of my homelab before choosing the architecture. In this chapter I recorded the available desktop hardware, Windows environment, network access, and the engineering decisions that followed from those facts.

## Why This Matters in a Company

Before deploying infrastructure, I need to know the actual capacity, platform state, and operational constraints of the host. This prevents me from designing around assumptions and gives me a verified baseline for later virtualization, networking, security, and automation work.

## Business Scenario

I am using the lab to simulate the infrastructure of a small organization with multiple user groups, centralized services, segmented networking, security controls, monitoring, backup, and later automation/cloud integration.

I want the environment to be realistic enough to demonstrate junior systems administration, network engineering, cloud, and DevOps skills while remaining practical on one physical workstation.

## Core Lab Constraint

I operate the entire required lab from a **single Windows desktop**.

My desktop serves two roles:

1. **Physical administrator workstation** — my normal Windows environment for managing, testing, documenting, and troubleshooting the lab.
2. **Lab compute host** — Hyper-V virtual machines and later network simulation/emulation provide the servers, clients, firewalls, routers, switches, and other infrastructure required by later chapters.

I intentionally excluded my separate laptop from the required architecture so the project remains convenient to reproduce and operate from one machine. I may only use it later as an optional extension if there is a clear engineering reason.

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
- [x] Identify why the CPU exposed only 12 logical processors although the installed model supports 24 threads
- [x] Re-enable SMT in Ryzen Master and verify that Windows exposes 24 logical processors
- [x] Identify the currently active Windows virtualization state
- [x] Confirm hardware virtualization support is enabled in firmware
- [x] Define the physical-host strategy: single desktop
- [x] Choose the virtualization platform: Hyper-V
- [x] Choose planned network simulation/emulation tooling: GNS3 with Hyper-V support
- [x] Define the first architecture diagram
- [x] Define naming conventions
- [x] Define an initial IP addressing plan
- [x] Record final Chapter 0 engineering decisions

## Hardware Inventory

### Desktop

I performed the initial PowerShell audit on 2026-08-12 and completed the host validation on 2026-08-13.

| Component | Observed value |
|---|---|
| Manufacturer | Micro-Star International Co., Ltd. |
| Motherboard | MSI B650 GAMING PLUS WIFI (MS-7E26), revision 1.0 |
| BIOS | American Megatrends / MSI BIOS 1.L0, dated 2025-06-19 |
| CPU | AMD Ryzen 9 7900 12-Core Processor |
| Physical CPU cores | 12 |
| Logical processors exposed to Windows | 24 — verified after SMT restoration |
| CPU hardware thread count | 24 |
| RAM | 31.1 GB usable/reportable |
| Storage | SPCC M.2 PCIe SSD, 953.87 GB |
| Architecture | 64-bit |
| Firmware virtualization | Enabled |
| BIOS SMT Control | Auto |
| Ryzen Master SMT state | ON after remediation |
| Windows edition | Windows 11 Pro |
| Windows release/build | 25H2, build 26200.8973 |

During the audit, the Ryzen 9 7900 initially appeared in Windows as 12 cores and 12 logical processors while WMI still reported `ThreadCount = 24`. I verified that no Windows `numproc` boot limit was present and that BIOS `SMT Control` was set to `Auto`.

I then inspected AMD Ryzen Master and found `Simultaneous Multithreading = OFF`. I changed SMT to `ON`, applied the change, and restarted Windows. After the restart, Windows correctly reported 12 cores, 24 logical processors, and 24 threads.

I resolved this host CPU-capacity anomaly before beginning the virtualization chapter.

### Installed CPU / Platform Utilities Relevant to Investigation

My Windows software inventory identified:

- AMD Ryzen Master 2.14.2.3341
- AMD Ryzen Master SDK 3.0.0.3620
- MSI Center SDK 3.2026.0410.01

Ryzen Master was the relevant control point for the logical-processor issue.

### Additional Physical Hardware

I do not require additional physical hardware for the core project.

## Network Environment

I observed these adapters during the initial audit:

| Adapter | State | Observed link speed / purpose |
|---|---|---|
| RZ616 Wi-Fi 6E 160MHz | Up | 144.4 Mbps; current host connectivity |
| Realtek Gaming 2.5GbE | Disconnected | Physical 2.5 GbE Ethernet adapter |
| Tailscale Tunnel | Up | Overlay/VPN adapter |
| VirtualBox Host-Only Ethernet Adapter | Up | 1 Gbps virtual host-only adapter |
| Additional VirtualBox Host-Only adapter | Not present | Stale/unused virtual adapter |

The VirtualBox adapters showed that another virtualization platform had previously been installed. I did not include them in the final lab architecture and can review them later if cleanup becomes useful.

## Windows Virtualization State at the Chapter 0 Audit

I verified this state from an elevated PowerShell session on 2026-08-12:

| Feature / setting | Observed state |
|---|---|
| Microsoft-Hyper-V-All | Disabled |
| VirtualMachinePlatform | Enabled |
| HypervisorPlatform | Disabled |
| `hypervisorlaunchtype` | Auto |
| Virtualization-Based Security | Enabled and running |
| Memory Integrity / HVCI | Configured and running |

This showed that the Microsoft hypervisor was already active for Windows security features even though I had not yet enabled the full Hyper-V VM-management feature. Chapter 1 later changed this state by enabling the full Hyper-V platform.

## Design Decisions

### DD-001 — Single-desktop lab

**Decision:** I will run all required project work from one Windows desktop using virtualization and, where useful, network simulation/emulation.

**Reasoning:** This removes the operational friction of moving between physical machines while still allowing me to build and test realistic isolated servers, clients, firewalls, and network topologies.

### DD-002 — Preserve the Windows security baseline

**Decision:** I will keep the host's VBS/Memory Integrity security baseline while building the lab.

**Reasoning:** I want the project to demonstrate sensible infrastructure practice on my everyday workstation rather than weakening the host simply to accommodate another virtualization stack.

### DD-003 — Hyper-V as the primary desktop hypervisor

**Decision:** I selected Microsoft Hyper-V as the primary virtualization platform.

**Reasoning:** My host runs Windows 11 Pro, firmware virtualization is enabled, the Microsoft hypervisor was already active for Windows security features, and Hyper-V integrates naturally with Windows administration and PowerShell.

**Consequence:** Chapter 1 enables and validates the full Hyper-V feature, establishes storage conventions, creates virtual networking, and deploys a small validation VM before the enterprise services are built.

### DD-004 — GNS3 for advanced network emulation

**Decision:** I will add GNS3 later when router/switch appliance emulation and larger network topologies become useful.

**Reasoning:** Hyper-V hosts the main server/client infrastructure, while GNS3 can add dedicated network-engineering capabilities without requiring a second physical machine.

### DD-005 — Restore full CPU thread capacity before virtualization

**Decision:** I resolved the 12-logical-processor anomaly before deploying the lab.

**Reasoning:** My desktop is the project's only required physical compute platform. Restoring and verifying the expected 24 logical processors prevents me from designing around an unexplained limitation and demonstrates proper pre-deployment troubleshooting.

### DD-006 — Segmented lab architecture behind FW01

**Decision:** I designed separate USERS, SERVERS, and MGMT networks behind `FW01`, with `10.10.0.0/16` reserved for the lab and functional `/24` subnets allocated from it.

**Reasoning:** This gives me meaningful routing and firewall boundaries while remaining simple enough to understand and reproduce on one desktop. I designed `vSW-USERS` and `vSW-SERVERS` as private Hyper-V switches and `vSW-MGMT` as internal so my Windows administrator workstation can reach the lab through the management network. I use the Hyper-V Default Switch as the planned upstream/WAN side for `FW01`.

I documented the detailed architecture and addressing in `docs/00-enterprise-architecture.md`.

## Evidence

### Host CPU and virtualization state

![Windows Task Manager CPU evidence](../screenshots/chapter-00/00-01-host-cpu.png)

This screenshot records my real host state after remediation: Ryzen 9 7900, 12 physical cores, 24 logical processors, and virtualization enabled.

### SMT remediation verification

![PowerShell SMT verification](../screenshots/chapter-00/00-03-smt-24-logical-processors.png)

This PowerShell verification confirms 12 cores, 24 logical processors, and 24 threads after I restored SMT.

The complete evidence inventory is tracked in [`screenshots/evidence-index.md`](../screenshots/evidence-index.md). I do not commit raw command output simply for completeness when it adds no useful portfolio evidence.

## Completion Criteria

I considered Chapter 0 complete when:

1. I documented the desktop hardware, Windows edition/build, and interfaces.
2. I confirmed virtualization capability and the Windows virtualization state.
3. I selected the virtualization platform and planned network-lab tooling with stated reasons.
4. I restored SMT and verified the expected 24 logical processors.
5. I designed the initial lab architecture, naming convention, and IP plan.
6. I could begin the implementation chapter without guessing about the environment.

**Status: COMPLETE.** I moved into Chapter 1 with a verified host baseline and documented architecture.