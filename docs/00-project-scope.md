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

- [ ] Record desktop hardware and Windows edition
- [ ] Record available network interfaces
- [ ] Record current connectivity constraints
- [ ] Confirm hardware virtualization support
- [x] Define the physical-host strategy: single desktop
- [ ] Choose the virtualization platform
- [ ] Choose network simulation/emulation tooling
- [ ] Define the first architecture diagram
- [ ] Define naming conventions
- [ ] Define an initial IP addressing plan
- [ ] Record Chapter 0 engineering decisions

## Hardware Inventory

### Desktop

To be measured.

### Additional Physical Hardware

Not required for the core project.

## Network Environment

To be measured.

## Design Decisions

### DD-001 — Single-desktop lab

**Decision:** All required project work will run from one Windows desktop using virtualization and, where useful, network simulation/emulation.

**Reasoning:** This removes the operational friction of moving between physical machines while still allowing realistic isolated servers, clients, firewalls, and network topologies to be built and tested.

**Consequence:** Desktop RAM, CPU virtualization support, available storage, and Windows edition become important design constraints. Resource allocation will need to be managed carefully when several VMs are running simultaneously.

No virtualization-platform decision is considered final until the desktop audit is complete.

## Evidence

Only relevant sanitized output/screenshots will be stored in this repository.

## Completion Criteria

Chapter 0 is complete when:

1. The desktop hardware, Windows edition, and interfaces are documented.
2. Virtualization capability is confirmed.
3. The virtualization platform and network-lab tooling have been selected with stated reasons.
4. An initial lab architecture has been designed.
5. The next implementation chapter can begin without guessing about the environment.
