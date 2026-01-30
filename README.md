# Proxmox → Kubernetes Foundations

Linux bare-metal lab using Proxmox as a Type-1 hypervisor and evolving into a
Kubernetes-ready cloud-native environment.

## Project Overview
This repository documents the design and implementation of a Linux-based
infrastructure lab built from bare metal. The project focuses on foundational
Linux skills such as virtualization, networking, and system administration,
with a progressive path toward Kubernetes and cloud-native workloads.

The approach is engineering-driven: architectural decisions, configuration
trade-offs, and troubleshooting steps are documented as part of the learning
process.

## Current Status
- Proxmox installed on bare-metal server
- Management network configured manually
- Direct laptop-to-server access for initial setup
- Networking troubleshooting and validation in progress

## Repository Structure
- `docs/` – Structured documentation (objectives, architecture, networking)
- `logs/` – Troubleshooting notes and lessons learned
- `diagrams/` – Architecture and network diagrams
- `scripts/` – Automation scripts (to be added incrementally)

## Next Steps
- Finalize Proxmox networking configuration
- Create base Linux virtual machines
- Prepare Kubernetes-ready VM images
- Document system evolution and decisions
