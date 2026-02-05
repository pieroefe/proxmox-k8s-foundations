# High-Level Architecture Overview

This diagram represents the **high-level architecture** of the Kubernetes-based Omada lab environment. It shows how users access the platform, how the Omada controllers are deployed inside the cluster, and how monitoring and observability are integrated.

---

## User Access and Edge Layer

Users access the Omada Controllers through DNS hostnames such as:

- `omada-v5.lab`
- `omada-v6.lab`

These hostnames resolve to the **Nginx Edge server** (`192.168.200.50`), which is deployed **outside the Kubernetes cluster**.

The Nginx Edge acts as a **reverse proxy and TLS termination point**, routing incoming HTTPS requests to the correct Omada Controller based on the requested hostname.

---

## Kubernetes Cluster

The core workloads run inside a **Kubernetes cluster** in the `10.10.10.0/24` network, composed of:

- One control-plane node (`k8s-master`)
- Two worker nodes (`k8s-worker1`, `k8s-worker2`)

Application components run as **pods**, which are dynamically scheduled across the worker nodes by Kubernetes. Pod placement is not fixed and may change over time.

---

## Omada Controller Workloads

Each Omada version runs in its **own dedicated namespace** to provide isolation and independent lifecycle management:

### Namespace: `omada-v5`
- Omada Controller v5 deployed as a Kubernetes Deployment
- Exposed internally via a ClusterIP Service (ports 8088 and 8043)
- Persistent configuration and state stored using PVC/PV

### Namespace: `omada-v6`
- Omada Controller v6 deployed as a Kubernetes Deployment
- Exposed internally via a ClusterIP Service (ports 8088 and 8043)
- Persistent data stored using dedicated PVC/PV resources

External access to these services is enabled through **NodePort exposure**, which allows the Nginx Edge to forward traffic from outside the cluster to the internal Kubernetes Services.

---

## Monitoring and Observability

Cluster monitoring is implemented in the **`monitoring` namespace** using a standard Kubernetes monitoring stack:

- **Prometheus** for metrics collection
- **Grafana** for visualization and dashboards
- **Alertmanager** for alert handling
- **kube-state-metrics** for Kubernetes object metrics
- **node-exporter** for node-level metrics

This monitoring stack operates independently from the Omada workloads while observing the entire cluster.

---

## Architectural Design Principles

- Logical separation using Kubernetes namespaces
- Externalized access layer via Nginx Edge
- Dynamic pod scheduling across worker nodes
- Persistent storage abstraction using PVC/PV
- Clear separation between application workloads and monitoring components

This architecture enables safe version comparison, scalability, and operational visibility within the lab environment.

