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

`FW01` routes and will enforce firewall policy between the isolated lab networks.

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

I verified the pre-router network baseline from both Windows and TEST01, then defined the BelkaCorp address plan across USERS (`10.10.10.0/24`), SERVERS (`10.10.20.0/24`), and MGMT (`10.10.30.0/24`). I documented the Layer-2 boundaries created by the Hyper-V virtual switches and the Layer-3 routing responsibilities that belong to `FW01`.

I also defined the `.1` gateway convention, routing and return-path behavior, Windows-specific routes that preserve the host's normal Wi-Fi default route, DHCP ownership on `DC01`, DHCP relay through `FW01`, and internal DNS ownership on `DC01` for the future Active Directory environment.

**Chapter 3 — In progress**

I deployed `FW01` as a Generation 2 Hyper-V VM and installed OPNsense 26.7. Before first boot I corrected the VM to 2 vCPU, fixed 4096 MB RAM, Secure Boot disabled, automatic checkpoints disabled, the dedicated VHDX, and four network adapters attached to the intended virtual switches.

The first post-install start fell through to Hyper-V PXE, so I investigated the boot chain instead of reinstalling. After explicitly prioritizing `FW01.vhdx` in the Generation 2 firmware boot order, OPNsense booted successfully from disk and reached the normal console menu.

I mapped every OPNsense guest interface to the authoritative Hyper-V adapter by matching MAC addresses. This verified `hn0=WAN`, `hn1=USERS`, `hn2=SERVERS`, and `hn3=MGMT`. I applied the OPNsense roles as `WAN=hn0`, `LAN=hn3` for management, `OPT1=hn1` for USERS, and `OPT2=hn2` for SERVERS.

I configured all three static BelkaCorp internal gateway addresses on FW01: USERS `10.10.10.1/24`, SERVERS `10.10.20.1/24`, and MGMT `10.10.30.1/24`. From the Windows host at `10.10.30.10/24`, I verified management connectivity and successfully opened the OPNsense HTTPS Web GUI at `https://10.10.30.1`.

I then verified OPNsense's routing foundation. The routing table contains directly connected routes for `10.10.10.0/24` on `hn1`, `10.10.20.0/24` on `hn2`, and `10.10.30.0/24` on `hn3`, plus a default route through WAN on `hn0`. Explicit route lookups selected the expected internal interfaces, FW01 successfully reached the Windows MGMT host, and a ping to public IP `1.1.1.1` completed with 0% packet loss.

Steps 3.3 through 3.6 are complete. The current task is **3.7 — add the Windows host's specific routes for USERS and SERVERS through `10.10.30.1` while preserving the normal Wi-Fi default route**.

## AI-Assisted Learning and Documentation

I use AI as a learning, troubleshooting, and documentation assistant during this project. I still execute and verify the infrastructure changes myself. The repository only records configurations, observations, troubleshooting incidents, and evidence that I have actually produced or verified in the lab.

## Portfolio Principle

This repository is evidence of work I have performed, not a collection of generated examples. I do not commit credentials, secrets, license keys, or sensitive configuration.
