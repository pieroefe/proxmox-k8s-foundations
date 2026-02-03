# Kubernetes Bootstrap (kubeadm) - Runbook with inline explanations
# Everything is inside this single bash code block so GitHub renders it cleanly.
# Notes are written as bash comments (#) so you can copy/paste safely.

###############################################################################
# Cluster Topology (reference)
###############################################################################
# k8s-master  (Control Plane)  10.10.10.10
# k8s-worker1 (Worker)         10.10.10.11
# k8s-worker2 (Worker)         10.10.10.12
#
# Convention used below:
# - "ALL NODES"   => run on k8s-master + all workers
# - "MASTER ONLY" => run only on k8s-master
# - "WORKERS ONLY"=> run only on k8s-worker1 and k8s-worker2
#
# Why this order matters:
# - kubeadm/kubelet require: swap OFF + sysctl + container runtime ready
# - admin.conf exists only AFTER kubeadm init (master)
# - nodes become Ready only AFTER installing a CNI (Flannel here)

###############################################################################
# 1) ALL NODES - Base dependencies
###############################################################################
# Why: needed to download repo keys securely (curl/ca-certificates) and manage
# keys/repositories (gpg/apt-transport-https).
sudo apt update
sudo apt install -y curl ca-certificates gpg apt-transport-https

###############################################################################
# 2) ALL NODES - Add Kubernetes repository (pkgs.k8s.io)
###############################################################################
# Why: Ubuntu default repos may not include kubeadm/kubelet/kubectl for the
# versions we need. Official Kubernetes packages are published via pkgs.k8s.io.

# Create a safe location for APT keyrings (recommended approach).
sudo mkdir -p /etc/apt/keyrings

# Import the repository signing key (v1.29 stable channel).
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes APT repository.
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list >/dev/null

# Refresh package lists so kubeadm/kubelet/kubectl become available.
sudo apt update

###############################################################################
# 3) ALL NODES - Install Kubernetes components
###############################################################################
# Why:
# - kubelet  = node agent (runs on every node)
# - kubeadm  = bootstrap/join tool (init on master, join on workers)
# - kubectl  = admin CLI (needed mainly on master, but installing everywhere is ok)
sudo apt install -y kubelet kubeadm kubectl

# Why hold:
# - Avoid surprise upgrades that may break cluster behavior during lab/testing.
sudo apt-mark hold kubelet kubeadm kubectl

###############################################################################
# 4) ALL NODES - Disable swap
###############################################################################
# Why: kubelet requires swap disabled by default. Leaving swap enabled can cause:
# - kubeadm init/join failures
# - kubelet refusing to start correctly
sudo swapoff -a

# Why: make swap-off persistent across reboots by commenting swap entries.
sudo sed -i '/ swap / s/^/#/' /etc/fstab

###############################################################################
# 5) ALL NODES - Kernel modules + sysctl for Kubernetes networking
###############################################################################
# Why:
# - overlay: common filesystem/overlay support used by container stacks
# - br_netfilter: allows iptables to see bridged traffic (required by many CNIs)
sudo modprobe overlay
sudo modprobe br_netfilter

# Why: persist kernel modules on reboot.
sudo tee /etc/modules-load.d/k8s.conf >/dev/null <<'EOF'
overlay
br_netfilter
EOF

# Why:
# - net.bridge.bridge-nf-call-iptables/ip6tables: make bridged traffic visible
#   to iptables rules (needed for pod networking)
# - net.ipv4.ip_forward: allow routing between pod networks
sudo tee /etc/sysctl.d/k8s.conf >/dev/null <<'EOF'
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

# Apply sysctl settings immediately.
sudo sysctl --system

###############################################################################
# 6) ALL NODES - Container runtime: containerd
###############################################################################
# Why: Kubernetes needs a container runtime to run pods. containerd is widely used.
sudo apt install -y containerd
sudo mkdir -p /etc/containerd

# Generate a default config so we can tune it (SystemdCgroup).
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null

# Why: kubelet on systemd-based distros works best with systemd cgroups enabled.
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Enable and restart containerd to load the new config.
sudo systemctl enable --now containerd
sudo systemctl restart containerd

###############################################################################
# 7) MASTER ONLY - Initialize control plane
###############################################################################
# Why: This creates the control-plane components + certs + kubeconfig (admin.conf).
# Also prints the "kubeadm join ..." command used by the workers.

# Sanity check: confirm the master has the expected static IP.
ip a | grep 10.10.10.10 -n

# Important networking choice:
# - apiserver-advertise-address must be reachable by workers (10.10.10.10)
# - pod-network-cidr must be a network that does NOT collide with your node subnet
#   (we use 10.244.0.0/16 for Flannel; your nodes are 10.10.10.0/24)
sudo kubeadm init \
  --apiserver-advertise-address=10.10.10.10 \
  --pod-network-cidr=10.244.0.0/16

###############################################################################
# 8) MASTER ONLY - Configure kubectl (kubeconfig)
###############################################################################
# Why: kubectl needs credentials + cluster endpoint to talk to the API server.
# The file /etc/kubernetes/admin.conf exists only after kubeadm init.
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Quick check: master node will likely be NotReady until CNI is installed.
kubectl get nodes -o wide

###############################################################################
# 9) MASTER ONLY - Install CNI (Flannel)
###############################################################################
# Why: Without a CNI, pods can't communicate and nodes often remain NotReady.
# Flannel is a simple default choice for a single L2 lab network.
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# Wait a bit and confirm kube-system pods are running (coredns, flannel).
kubectl get pods -n kube-system
kubectl get nodes -o wide

###############################################################################
# 10) WORKERS ONLY - Join cluster (run on each worker with sudo)
###############################################################################
# Why: Workers must register to the control plane. This writes system configs under
# /etc/kubernetes and starts services, so it MUST be run as root (sudo).
#
# IMPORTANT:
# - Do NOT paste the "Then you can join..." text. Paste ONLY the join command.
# - Use the exact token/hash printed by kubeadm init on the master.
#
# Option A (recommended): generate a fresh join command on the master if needed:
#   sudo kubeadm token create --print-join-command
#
# Option B: use the join line you already have, but prepend sudo.
#
# Example template (replace <TOKEN> and <HASH>):
# sudo kubeadm join 10.10.10.10:6443 --token <TOKEN> \
#   --discovery-token-ca-cert-hash sha256:<HASH>

###############################################################################
# 11) MASTER ONLY - Validation after workers join
###############################################################################
# Why: confirm all nodes registered and are Ready, and core services are healthy.
kubectl get nodes -o wide
kubectl get pods -n kube-system

###############################################################################
# Troubleshooting quick checks (ALL NODES) - run only if something fails
###############################################################################
# Check swap status (should be empty):
# sudo swapon --show
#
# Check containerd status:
# systemctl status containerd --no-pager
#
# Check kubelet status:
# systemctl status kubelet --no-pager
#
# Get kubeadm preflight details (last resort when join/init fails):
# sudo kubeadm join ... --v=5
