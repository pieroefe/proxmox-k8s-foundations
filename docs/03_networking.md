# Networking Design and Configuration

This document describes the networking design used in the infrastructure lab, focusing on Linux networking fundamentals and their integration with Proxmox virtualization and Kubernetes preparation.

A clear and predictable networking setup is essential, as it provides management access to the hypervisor, enables reliable communication between virtual machines, and serves as the foundation for the Kubernetes cluster.

---

## Networking Goals

The networking layer is designed to meet the following objectives:

- Provide stable management access to the Proxmox host
- Enable reliable connectivity between virtual machines
- Keep the initial setup simple and easy to troubleshoot
- Expose and reinforce core Linux networking concepts
- Prepare the environment for Kubernetes node communication

---

## Management Network Design

A dedicated management network is used to access the Proxmox web interface and perform administrative tasks.

Key characteristics of this network include:

- Static IP addressing
- Direct connectivity during the initial setup phase
- Clear separation between management access and future workload traffic

This approach reduces external dependencies and simplifies early-stage troubleshooting.

---

## Linux Bridge Configuration (vmbr0)

Proxmox relies on Linux bridges to connect virtual machines to physical network interfaces. In this lab, the primary bridge used is `vmbr0`.

The bridge is responsible for:

- Hosting the management IP address of the Proxmox node
- Connecting virtual machines to the physical network
- Abstracting the physical interface from higher layers

A key design decision is assigning the management IP address to the bridge (`vmbr0`) rather than to the physical network interface. This ensures network connectivity remains consistent as virtual machines are added or modified.

---

## Physical Interface Binding

The physical network interface is configured solely as a bridge port and does not hold an IP address.

Its role is limited to forwarding traffic between the Linux bridge and the physical network, which allows:

- Cleaner network abstraction
- Reduced risk of misconfiguration
- Alignment with Proxmox networking best practices

---

## Initial Connectivity Model

During the initial deployment phase, direct connectivity between a laptop and the server is used. Static IP addresses are manually configured on both ends to establish management access without relying on external services such as DHCP.

This model enables:

- Fast validation of basic connectivity
- Easier isolation of network-related issues
- Clear visibility into Linux interface behavior

---

## Virtual Machine Networking (Kubernetes Preparation)

As the project progresses beyond basic virtualization, networking considerations are extended to the Linux virtual machines that will act as Kubernetes nodes.

Kubernetes places strict requirements on node reachability and network stability. For this reason, networking inside the virtual machines is treated as a deliberate design choice rather than a default configuration.

---

## IP Addressing Strategy for Kubernetes Nodes

All Kubernetes nodes use static IP addressing configured at the operating system level.

This decision ensures:

- Stable node identity within the cluster
- Predictable control-plane and worker communication
- Independence from DHCP during cluster initialization
- Simplified troubleshooting

Dynamic IP addressing was intentionally avoided to prevent issues during cluster bootstrap and node joins.

---

## Node Addressing Scheme

| Node Name    | Role          | IP Address  |
|-------------|---------------|-------------|
| k8s-master  | Control Plane | 10.10.10.10 |
| k8s-worker1 | Worker Node   | 10.10.10.11 |
| k8s-worker2 | Worker Node   | 10.10.10.12 |

All nodes reside on the same Layer 2 network provided by the Proxmox bridge (`vmbr0`), enabling direct east–west communication without additional routing complexity.

---

## Guest Networking Configuration

Inside each virtual machine, networking is configured using Netplan with the following principles:

- Static IPv4 configuration
- Explicit default gateway
- Manually defined DNS resolvers
- Single primary network interface

This configuration provides deterministic behavior and aligns with Kubernetes initialization requirements.

---

## Host-to-Guest and Guest-to-Guest Connectivity

The Linux bridge (`vmbr0`) serves as the Layer 2 domain for the Proxmox host and all Kubernetes nodes.

This allows:

- Management access to the hypervisor
- Direct communication between Kubernetes nodes
- A simple and observable networking model during early cluster stages

At this phase, no VLAN segmentation or advanced network isolation is implemented. These features are intentionally deferred until the base cluster is stable.

---

## Common Networking Pitfalls Encountered

During setup, several issues were encountered and documented:

- Assigning the management IP to the physical interface instead of the bridge
- Misidentifying Linux network interface names
- Network changes not taking effect due to missing reloads
- Confusion between host networking and guest networking

These issues highlight the importance of understanding Linux networking concepts rather than relying solely on graphical interfaces.

---

## Future Networking Evolution

Once the Kubernetes cluster is fully operational, the networking layer may be extended to include:

- VLAN-based segmentation
- Separation between management and workload traffic
- Kubernetes CNI-specific considerations
- Integration with external switching and routing infrastructure

All future enhancements will build on the same Linux bridge fundamentals introduced in this phase.
