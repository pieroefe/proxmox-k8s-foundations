# Proxmox Installation and Initial Setup

This document describes the installation and initial configuration of Proxmox
VE on bare-metal hardware. The objective of this phase is to establish a stable,
accessible, and well-understood virtualization environment that serves as the
foundation for all subsequent infrastructure layers.

Rather than providing a step-by-step tutorial, this document focuses on the key
decisions, validation steps, and operational considerations involved in the
installation process.

## Installation Context

Proxmox is installed directly on bare-metal hardware to operate as a Type-1
hypervisor. This design choice avoids the overhead and complexity of nested
virtualization and provides direct access to system resources such as CPU,
memory, storage, and network interfaces.

The installation is performed in an isolated environment to allow full control
over networking and system configuration during the initial setup phase,
reducing external dependencies and simplifying troubleshooting.

## Base Installation

The base installation includes:
- Proxmox VE installed from the official ISO image
- Default Linux kernel and Proxmox components
- Initial system configuration performed during installation

At this stage, no virtual machines are created. The primary goal is to ensure
that the hypervisor itself is stable, reachable, and correctly configured before
introducing additional complexity.

## Initial Network Configuration

After installation, the most critical task is configuring network access to the
Proxmox management interface.

The following principles guide the initial network configuration:
- The management IP address is assigned to a Linux bridge (`vmbr0`)
- The physical network interface is attached to the bridge as a port
- Static IP addressing is used during the initial setup phase

Assigning the management IP to the bridge rather than to the physical interface
ensures consistent connectivity as virtual machines are added and the network
topology evolves.

## Management Access Validation

Once networking is configured, management access is validated through multiple
checks. Connectivity is verified at the Linux level before attempting access to
the Proxmox web interface.

Successful access over HTTPS confirms that the virtualization layer is
operational and ready to host virtual machines.

## Post-Installation Checks

After establishing management access, several baseline checks are performed to
confirm system readiness:
- Verification of detected system resources (CPU, memory, storage)
- Review of Linux bridge configuration
- Confirmation that Proxmox core services are running as expected

These checks establish a known-good baseline prior to virtual machine creation
or the introduction of orchestration layers.

## Common Issues Encountered

During installation and initial setup, several issues were encountered and
resolved, including:
- Loss of management access due to incorrect IP assignment
- Network configuration changes not being applied correctly
- Confusion between physical network interfaces and Linux bridges

These issues are documented in the troubleshooting logs and directly influenced
subsequent configuration and validation practices.

## Useful Commands and Validation Steps

The following commands were used throughout the installation and validation
process as diagnostic tools to observe system behavior and confirm configuration
changes.

The commands were not executed as a fixed sequence, but selected based on the
state of the system and the issue being investigated.

To inspect network interfaces and confirm correct IP assignment to the Linux
bridge:

ip a
To identify physical network interface names and verify their operational state:

ip link
To validate local routing and confirm correct subnet association:

ip route
To apply network configuration changes without rebooting the system:

ifreload -a
As an alternative method to reload networking services when required:

systemctl restart networking
To test basic network connectivity between the Proxmox host and the management
client:

ping <management-ip>
To verify that the Proxmox web interface service is running:

systemctl status pveproxy
To confirm that core Proxmox management services are active:

systemctl status pvedaemon