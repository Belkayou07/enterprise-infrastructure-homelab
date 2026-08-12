# Enterprise Infrastructure Homelab

A hands-on infrastructure engineering portfolio project built to simulate a small business environment and develop practical skills in systems administration, networking, security, automation, cloud, and DevOps.

## Project Goal

Design, deploy, secure, operate, troubleshoot, document, and progressively automate a realistic enterprise-style lab using physical hardware and virtual machines.

The project is intentionally built in chapters. Each chapter must be working and understood before the next one is treated as complete.

## Learning Path

1. **Chapter 0 — Scope, hardware audit, and repository setup**
2. **Chapter 1 — Virtualization platform**
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

The exact implementation will be selected during the relevant chapters rather than assumed in advance.

- Virtualization
- Windows Server
- Linux
- Active Directory
- DNS and DHCP
- Routing and VLANs
- Firewalling
- Monitoring
- Backup and recovery
- Git and GitHub
- PowerShell and Bash
- Ansible
- Terraform
- Azure
- Containers and CI/CD

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

**Chapter 0 — In progress**

Next: inventory the available laptop and desktop hardware, document the current network situation, and decide the virtualization architecture.

## Portfolio Principle

This repository is evidence of work performed, not a collection of generated examples. Credentials, secrets, private IP information that should not be public, license keys, and sensitive configuration will never be committed.
