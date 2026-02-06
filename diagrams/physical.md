# Physical Architecture

This section describes the **physical infrastructure** supporting the lab environment, including network devices, switching, compute, and virtualization layers.

---

## Edge Routing and Network Access

- **MikroTik CCR2116-12G-4S+**
  - Acts as the edge router and Internet gateway
  - Provides WAN connectivity, NAT, and inter-VLAN routing

- **TP-Link TL-SG2008P**
  - Operates as the access switch
  - Hosts **VLAN 200**, used for lab and management traffic
  - Connects the physical server and the user workstation

---

## Compute and Virtualization Layer

- **Physical Server (Bare Metal)**
  - Hosts the virtualization platform
  - Connected to the switch on VLAN 200

- **Proxmox VE**
  - Provides the virtualization layer for all lab workloads
  - Abstracts physical hardware from virtual machines

---

## Virtual Machines

The following virtual machines run on Proxmox:

- **nginx-edge**
  - IP: `192.168.200.50`
  - Acts as reverse proxy and TLS termination point

- **k8s-master**
  - IP: `10.10.10.10`
  - Kubernetes control-plane node

- **k8s-worker1**
  - IP: `10.10.10.11`
  - Kubernetes worker node

- **k8s-worker2**
  - IP: `10.10.10.12`
  - Kubernetes worker node

---

## Design Notes

- Network segmentation is achieved using VLAN 200
- Virtualization is used to isolate and scale workloads
- Kubernetes nodes are deployed as virtual machines rather than bare-metal hosts
