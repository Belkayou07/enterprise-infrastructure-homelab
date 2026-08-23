# Enterprise Infrastructure Homelab

This is my hands-on infrastructure engineering portfolio project. I am using it to simulate a small business environment and build practical skills in systems administration, networking, security, automation, cloud, and DevOps.

## Project Goal

I am designing, deploying, securing, operating, troubleshooting, documenting, and progressively automating a realistic enterprise-style lab entirely from a **single Windows desktop**.

My physical desktop remains my normal administrator workstation while virtual machines and network simulation/emulation provide the servers, clients, firewalls, routers, switches, and isolated networks used throughout the project.

I am intentionally building the project in chapters. I do not treat a chapter as complete until I have implemented, verified, understood, and documented the relevant work.

## Learning Path

1. **Chapter 0 — Scope, desktop audit, and repository setup** ✅
2. **Chapter 1 — Virtualization platform** ✅
3. **Chapter 2 — Network architecture and IP design** ✅
4. **Chapter 3 — Firewall and routing** ← current
5. **Chapter 4 — Windows Server and Active Directory**
6. **Chapter 5 — Linux server administration**
7. **Chapter 6 — Network segmentation and access control**
8. **Chapter 7 — Security hardening**
9. **Chapter 8 — Monitoring and observability**
10. **Chapter 9 — Backup and disaster recovery**
11. **Chapter 10 — Infrastructure automation**
12. **Chapter 11 — Cloud / DevOps extension and portfolio finalization**

## Planned Technology Areas

I select the exact implementation during the relevant chapters rather than assuming every tool in advance.

- Hyper-V virtualization
- Windows Server
- Linux
- Active Directory
- DNS and DHCP
- Routing and VLANs
- OPNsense firewalling
- GNS3 network simulation/emulation
- Monitoring
- Backup and recovery
- Git and GitHub
- PowerShell and Bash
- Ansible
- Terraform
- Azure
- Containers and CI/CD

## Logical Architecture

```text
Windows 11 Pro Desktop
│
├── Normal administrator workstation
│   ├── Browser / administration tools
│   ├── PowerShell / terminal
│   └── Git / GitHub
│
└── Hyper-V
    ├── Default Switch ── FW01 WAN
    ├── vSW-USERS   ──── CLIENT01
    ├── vSW-SERVERS ──── DC01 / LNX01 / MON01
    └── vSW-MGMT    ──── Windows host management path
```

`FW01` will route and enforce firewall policy between the isolated lab networks.

## Repository Structure

```text
enterprise-infrastructure-homelab/
├── README.md
├── docs/              # Chapter documentation and engineering decisions
├── screenshots/       # Selected evidence of completed work
└── troubleshooting/   # Incident and troubleshooting records
```

I only keep directories when they contain real project material. Later chapters can introduce configuration, diagram, or automation directories when those artifacts actually exist.

## Current Status

**Chapter 0 — Complete**

I audited the Windows host, restored SMT so the CPU exposes the expected 24 logical processors, selected Hyper-V as the primary hypervisor, and documented the initial BelkaCorp architecture, naming convention, VM resource budget, subnet plan, and virtual-switch design.

**Chapter 1 — Complete**

I enabled and verified Hyper-V, configured dedicated VM/VHDX storage, built the planned virtual switches, and configured the Windows host management adapter as `10.10.30.10/24` without a default gateway.

I created `TEST01` as a Generation 2 Ubuntu Server validation VM, installed Ubuntu Server, configured the guest as `10.10.30.20/24` on `vSW-MGMT`, and verified CPU, Dynamic Memory, virtual storage, networking, Secure Boot, persistence, and host-to-guest connectivity.

The chapter produced four real troubleshooting records: accidental duplicate Hyper-V switch objects, one-way ICMP connectivity caused by the Windows host firewall, an overly low TEST01 Dynamic Memory minimum, and an automatic-checkpoint chain detected during the final audit.

For the Dynamic Memory incident, I traced the low guest-memory reading to a 512 MB minimum and changed the policy to 2048 MB startup / 1536 MB minimum / 4096 MB maximum. For the checkpoint incident, I disabled automatic checkpoints, removed the existing checkpoints through Hyper-V, verified that TEST01 returned to direct `TEST01.vhdx` attachment with no remaining `.avhdx` files, and then successfully booted and reached the guest again.

The final Chapter 1 audit confirmed the Hyper-V feature and service state, BelkaCorp storage paths, virtual-switch inventory, `10.10.30.10/24` host management addressing, TEST01 configuration, clean checkpoint policy, base-VHDX attachment, and successful connectivity to `10.10.30.20`.

**Chapter 2 — Complete**

I verified the pre-router network baseline from both Windows and TEST01, then defined the BelkaCorp address plan across USERS (`10.10.10.0/24`), SERVERS (`10.10.20.0/24`), and MGMT (`10.10.30.0/24`). I documented the Layer-2 boundaries created by the Hyper-V virtual switches and the Layer-3 routing responsibilities that will belong to `FW01`.

I also defined the planned `.1` gateway convention, routing and return-path behavior, Windows-specific routes that will preserve the host's normal Wi-Fi default route, DHCP ownership on `DC01`, DHCP relay through `FW01`, and internal DNS ownership on `DC01` for the future Active Directory environment.

The final implementation plan provides a dependency-aware handoff into Chapter 3: deploy `FW01`, identify and configure its WAN/USERS/SERVERS/MGMT interfaces, verify the MGMT interface first, and only then introduce cross-subnet routes and later services.

**Chapter 3 — In progress**

I verified the pre-deployment Hyper-V state, downloaded and SHA-256 verified the OPNsense 26.7 `amd64` DVD installer, extracted the ISO into the Hyper-V ISO directory, and created the initial `FW01` Generation 2 VM.

Before the first boot, I audited and corrected the VM configuration. The final verified pre-boot state is 2 vCPU, fixed 4096 MB RAM, Secure Boot disabled, automatic checkpoints disabled, the correct VHDX and installer ISO attached, and four named adapters mapped to the Default Switch, `vSW-USERS`, `vSW-SERVERS`, and `vSW-MGMT`.

I installed OPNsense to `FW01.vhdx` and detached the installer ISO. The first post-install boot fell through to PXE instead of starting from the virtual disk, so I am currently validating and correcting the Hyper-V firmware boot order before declaring the installation complete.

## AI-Assisted Learning and Documentation

I use AI as a learning, troubleshooting, and documentation assistant during this project. I still execute and verify the infrastructure changes myself. The repository only records configurations, observations, troubleshooting incidents, and evidence that I have actually produced or verified in the lab.

## Portfolio Principle

This repository is evidence of work I have performed, not a collection of generated examples. I do not commit credentials, secrets, license keys, or sensitive configuration.
