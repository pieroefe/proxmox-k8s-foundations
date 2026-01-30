# Networking Design and Configuration

This document describes the networking design used in the infrastructure lab,
with a focus on Linux networking fundamentals and how they integrate with
Proxmox virtualization.

Reliable and well-understood networking is critical, as it provides management
access to the hypervisor and enables communication between virtual machines and
higher-level services.

## Networking Goals

The networking layer is designed to achieve the following goals:

- Provide stable management access to the Proxmox host
- Enable connectivity between the host and virtual machines
- Maintain simplicity during the initial setup phase
- Expose Linux networking concepts such as bridges and interfaces

## Management Network Design

A dedicated management network is used for accessing the Proxmox web interface
and administrative services.

Key characteristics:
- Static IP addressing
- Direct connectivity during initial setup
- Clear separation between management access and future workload traffic

This approach simplifies troubleshooting and reduces external dependencies
during early deployment stages.

## Linux Bridge (vmbr0)

Proxmox relies on Linux bridges to connect virtual machines to physical network
interfaces. In this lab, the primary bridge is `vmbr0`.

The bridge serves as the central networking component and is responsible for:
- Hosting the management IP address of the Proxmox node
- Connecting virtual machines to the physical network
- Abstracting the physical interface from higher layers

A key design decision is assigning the management IP address to the bridge
(`vmbr0`) rather than to the physical network interface. This ensures that
network connectivity remains consistent even as virtual machines are added or
network configurations evolve.

## Physical Interface Binding

The physical network interface is configured as a bridge port and does not hold
an IP address directly. Its role is limited to forwarding traffic between the
bridge and the physical network.

This separation allows:
- Cleaner network abstraction
- Reduced risk of misconfiguration
- Better alignment with Proxmox networking best practices

## Initial Connectivity Model

During the initial setup phase, direct connectivity between a laptop and the
server is used. Static IP addresses are manually configured on both ends to
establish management access without relying on external network services such
as DHCP.

This model enables:
- Faster validation of basic connectivity
- Easier isolation of network issues
- Clear visibility into Linux interface behavior

## Common Networking Pitfalls

Several common issues were encountered and documented during setup:

- Assigning the management IP to the physical interface instead of the bridge
- Misidentifying Linux network interface names
- Changes not taking effect due to missing network reloads
- Confusion between host networking and virtual machine networking

These issues reinforce the importance of understanding Linux networking
constructs rather than relying solely on graphical interfaces.

## Future Networking Evolution

As the project evolves, the networking layer will be extended to include:

- Integration with external switches and routers
- VLAN-based network segmentation
- Separation between management and workload traffic
- Networking considerations for Kubernetes clusters

These enhancements will build on the same Linux bridge concepts introduced in
this initial phase.

---

This networking design establishes a clear and stable foundation for both
virtualization and container orchestration while reinforcing essential Linux
networking principles.
