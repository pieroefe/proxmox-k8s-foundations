# Choice of Prometheus and Grafana

Prometheus was selected as the metrics backend because it is the native and most widely adopted monitoring system in Kubernetes environments. Its pull-based model and tight integration with Kubernetes components such as the kubelet, node-exporter, and kube-state-metrics allow metrics to be collected without additional adapters or custom instrumentation. This makes Prometheus a natural choice for observing cluster behavior in a predictable and reproducible way.

Grafana was chosen as the visualization layer due to its strong integration with Prometheus and its ability to present time-series metrics clearly at different levels, including nodes, namespaces, and pods. Grafana does not collect metrics itself; instead, it queries Prometheus and focuses exclusively on visualization, which enforces a clear separation of responsibilities between data collection and presentation.

Together, Prometheus and Grafana provide a lightweight but production-relevant observability stack. Their widespread adoption in cloud-native environments and alignment with CNCF best practices make them suitable for both operational validation in this project and as a representative monitoring solution for real-world Kubernetes deployments.

# Prometheus and Grafana Installation (kube-prometheus-stack)

This section documents the installation of Prometheus and Grafana on the Kubernetes cluster using the `kube-prometheus-stack` Helm chart. The objective is to establish basic observability before deploying application workloads.

All commands were executed from the Kubernetes control-plane node with `kubectl` access.

The first step is to install Helm, which is required to deploy the monitoring stack as a Helm chart. Helm simplifies the installation and lifecycle management of complex Kubernetes applications.

### curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
### helm version

Next, a dedicated namespace is created to isolate monitoring components from application workloads. This improves organization and avoids mixing operational services with user deployments.

### kubectl create namespace monitoring
### kubectl get namespaces

The Prometheus Community Helm repository is then added. This repository contains the officially maintained kube-prometheus-stack chart and related updates.

### helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
### helm repo update

With Helm and the repository configured, the monitoring stack is installed into the monitoring namespace. This single command deploys Prometheus, Grafana, Alertmanager, exporters, and prebuilt dashboards.

### helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring

After deployment, the status of the monitoring components is validated by checking the pods in the monitoring namespace. All pods should transition to the Running state.

### kubectl get pods -n monitoring

Grafana is accessed using kubectl port-forward, which provides temporary and secure access without exposing the service to the network. This approach is suitable for lab and validation environments.

### kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring

Grafana requires an initial admin password, which is stored in a Kubernetes secret created during installation. The password is retrieved and decoded as follows.

### kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode && echo

Once logged in, Grafana is already configured with Prometheus as its data source. Prometheus runs as a separate service in the cluster and provides metrics that Grafana queries and visualizes through dashboards.

To confirm that Prometheus is actively collecting metrics, the preinstalled dashboards can be used. Dashboards such as Node Exporter / Nodes and Kubernetes / Compute Resources / Namespace (Pods) provide immediate visibility into node health and pod resource usage.

At the end of this process, the cluster has a functional observability layer that enables real-time monitoring of nodes and workloads, providing a stable foundation for deploying and evaluating application services.








