# Enterprise Infrastructure Homelab

A hands-on infrastructure engineering portfolio project built to simulate a small business environment and develop practical skills in systems administration, networking, security, automation, cloud, and DevOps.

## Project Goal

Design, deploy, secure, operate, troubleshoot, document, and progressively automate a realistic enterprise-style lab entirely from a **single Windows desktop**.

The physical desktop remains the normal administrator workstation while virtual machines and network simulation/emulation provide the servers, clients, firewalls, routers, switches, and isolated networks used throughout the project.

The project is intentionally built in chapters. Each chapter must be working and understood before the next one is treated as complete.

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

The exact implementation is selected during the relevant chapters rather than assumed in advance.

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

Directories are added only when they contain real project material; empty folders are not kept simply for appearance.

## Current Status

**Chapter 0 — Complete**

The Windows host has been audited, SMT restored to expose the expected 24 logical processors, Hyper-V selected as the primary hypervisor, and the initial BelkaCorp architecture, naming convention, VM resource budget, subnet plan, and virtual-switch design have been documented.

**Chapter 1 — In progress**

Next: enable the full Hyper-V Windows feature, restart, verify the Hyper-V platform and management tools, then establish the VM storage and virtual-switch configuration.

## Portfolio Principle

This repository is evidence of work performed, not a collection of generated examples. Credentials, secrets, license keys, and sensitive configuration will never be committed.
