# Architecture Design

This document describes the architectural design of the infrastructure lab,
highlighting how each layer contributes to isolation, flexibility, and
operational clarity.

The architecture is intentionally layered to separate responsibilities between
hardware, virtualization, operating systems, and orchestration platforms.

## High-Level Architecture

The infrastructure follows a layered model:

1. Physical server (bare metal)
2. Proxmox VE (Type-1 hypervisor)
3. Linux virtual machines
4. Kubernetes orchestration layer
5. Containerized management services

Each layer has a clearly defined role and boundary, reducing complexity and
allowing individual components to evolve independently.

## Layer 1: Bare-Metal Hardware

The physical server provides the compute, memory, and storage resources for the
entire environment. Running Proxmox directly on bare metal eliminates the
overhead of nested virtualization and provides direct access to hardware
features such as networking and storage controllers.

This approach mirrors real-world infrastructure used in on-premise and edge
environments.

## Layer 2: Proxmox VE (Virtualization Layer)

Proxmox acts as a Type-1 hypervisor built on Linux, responsible for:

- Resource allocation and isolation
- Virtual machine lifecycle management
- Network bridging between physical and virtual interfaces
- Providing a stable foundation for guest operating systems

Using Proxmox allows centralized management of virtual machines while retaining
full control over the underlying Linux system.

## Layer 3: Linux Virtual Machines

Linux virtual machines serve as standardized execution environments for higher
layers. This abstraction provides:

- Consistent operating system behavior
- Isolation between services and workloads
- Flexibility to create templates and reusable images

Each VM can be tailored to a specific role, such as Kubernetes control plane,
worker nodes, or supporting infrastructure services.

## Layer 4: Kubernetes Orchestration

Kubernetes is introduced to manage containerized services running on top of the
Linux virtual machines. Its role in the architecture includes:

- Orchestrating multiple instances of management software
- Enforcing isolation between different software versions
- Managing service lifecycle and scaling
- Providing a consistent deployment model across environments

Kubernetes is not treated as a replacement for virtualization, but as a
complementary layer focused on application orchestration.

## Layer 5: Management and Support Services

At the top of the stack, containerized services represent the actual management
platforms and supporting tools. These may include:

- Network device management software
- Monitoring and observability tools
- Supporting services required for testing and validation

This layered approach allows multiple versions of management software to coexist
while remaining isolated and reproducible.

## Design Considerations

Several key design considerations influenced this architecture:

- **Isolation**: services and versions must not interfere with each other
- **Reproducibility**: environments should be easy to recreate
- **Flexibility**: infrastructure should adapt to changing requirements
- **Operational visibility**: issues should be observable and diagnosable

## Architectural Trade-Offs

The chosen architecture involves trade-offs, including:

- Increased complexity compared to single-VM setups
- Additional resource overhead introduced by virtualization layers
- Greater initial setup effort

These trade-offs are accepted in exchange for improved control, clarity, and
long-term scalability.

---

This architecture provides a stable and extensible foundation for managing
complex Linux-based software environments in a controlled and reproducible
manner.
