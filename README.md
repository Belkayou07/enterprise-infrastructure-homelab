# Enterprise Infrastructure Homelab

This is my hands-on infrastructure engineering portfolio project. I am using it to simulate a small business environment and build practical skills in systems administration, networking, security, automation, cloud, and DevOps.

## Project Goal

I am designing, deploying, securing, operating, troubleshooting, documenting, and progressively automating a realistic enterprise-style lab entirely from a **single Windows desktop**.

My physical desktop remains my normal administrator workstation while virtual machines and network simulation/emulation provide the servers, clients, firewalls, routers, switches, and isolated networks used throughout the project.

I am intentionally building the project in chapters. I do not treat a chapter as complete until I have implemented, verified, understood, and documented the relevant work.

## Learning Path

1. **Chapter 0 — Scope, desktop audit, and repository setup** ✅
2. **Chapter 1 — Virtualization platform** ← current
3. **Chapter 2 — Network architecture and IP design**
4. **Chapter 3 — Firewall and routing**
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
├── diagrams/          # Architecture and network diagrams
├── configs/           # Sanitized configuration exports/examples
├── scripts/           # Automation created during later chapters
├── screenshots/       # Selected evidence of completed work
└── troubleshooting/   # Incident and troubleshooting records
```

I only keep directories when they contain real project material; I do not add empty folders simply for appearance.

## Current Status

**Chapter 0 — Complete**

I audited the Windows host, restored SMT so the CPU exposes the expected 24 logical processors, selected Hyper-V as the primary hypervisor, and documented the initial BelkaCorp architecture, naming convention, VM resource budget, subnet plan, and virtual-switch design.

**Chapter 1 — Final audit in progress**

I enabled and verified Hyper-V, configured dedicated VM/VHDX storage, built the planned virtual switches, and configured the Windows host management adapter as `10.10.30.10/24` without a default gateway.

I created `TEST01` as a Generation 2 Ubuntu Server validation VM, verified its CPU, memory, VHDX, virtual switch attachment, and Linux-compatible Secure Boot configuration, installed Ubuntu Server, and configured the guest as `10.10.30.20/24` on `vSW-MGMT`.

I verified host-to-guest connectivity in both directions and proved that the static TEST01 Netplan configuration survives a shutdown/start cycle.

Chapter 1 has produced three real troubleshooting records: accidental duplicate Hyper-V switch objects, one-way ICMP connectivity caused by the Windows host firewall, and an overly low TEST01 Dynamic Memory minimum. For the memory incident, I traced a low guest-memory reading to Hyper-V Dynamic Memory, changed the policy to 2048 MB startup / 1536 MB minimum / 4096 MB maximum, and verified the corrected limits and persistent guest networking after a cold boot.

The remaining Chapter 1 task is a final consolidated audit of the host, storage paths, virtual switches, TEST01 state, management addressing, and evidence set before I mark the virtualization chapter complete.

## AI-Assisted Learning and Documentation

I use AI as a learning, troubleshooting, and documentation assistant during this project. I still execute and verify the infrastructure changes myself. The repository only records configurations, observations, troubleshooting incidents, and evidence that I have actually produced or verified in the lab.

## Portfolio Principle

This repository is evidence of work I have performed, not a collection of generated examples. I do not commit credentials, secrets, license keys, or sensitive configuration.
