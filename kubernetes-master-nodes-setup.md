# Kubernetes HA Cluster Setup (v1.33) with Calico

**Master IPs:** `192.168.73.21`, `192.168.73.22`, `192.168.73.23` | **VIP:** `192.168.73.10` | **Pod CIDR:** `10.244.0.0/16`

---

## Step 1 — Preparation (All Nodes)

Run this on **every node** — Masters and Workers.

```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/^\s*[^#]*swap\s/ s/^/#/' /etc/fstab

# Configure /etc/hosts
echo -e "192.168.73.21 k8s-m01\n192.168.73.22 k8s-m02\n192.168.73.23 k8s-m03\n192.168.73.10 k8s-vip" | sudo tee -a /etc/hosts > /dev/null

# Reboot
sudo reboot
```

> ⚠️ Swap must be disabled — Kubernetes will refuse to start if swap is active.

---

## Step 2 — Install Containerd and Kubernetes v1.33 (All Nodes)

```bash
# Dependencies and repo keys
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install packages
sudo apt-get update
sudo apt-get install -y containerd kubelet kubeadm kubectl
sudo apt-mark hold containerd kubelet kubeadm kubectl
```

> 💡 `apt-mark hold` prevents automatic upgrades — important for cluster stability.

```bash
# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay && sudo modprobe br_netfilter

# Configure sysctl for Kubernetes networking
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# Configure Containerd with SystemdCgroup
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

> ⚠️ `SystemdCgroup = true` is required for Kubernetes v1.33. Without this, the kubelet will crash-loop.

---

## Step 3 — Initialize the Cluster (k8s-m01 only)

Run this **only on the first Master node**:

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.73.21 \
  --control-plane-endpoint="192.168.73.10:6443" \
  --upload-certs \
  --pod-network-cidr=10.244.0.0/16
```

Once complete, set up kubectl access:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 💡 The `kubeadm init` output will print **two join commands** — save them both:
> - One for **Master nodes** (includes `--control-plane` and `--certificate-key`)
> - One for **Worker nodes** (standard join)
>
> If you lose them, regenerate with:
> ```bash
> # Regenerate worker join command
> kubeadm token create --print-join-command
>
> # Regenerate certificate key for master join
> kubeadm init phase upload-certs --upload-certs
> ```

---

## Step 4 — Install Calico via Tigera Operator (k8s-m01 only)

```bash
# Install the Tigera operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml

# Create the custom resources file
cat <<EOF > custom-resources.yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
---
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
EOF

# Apply the configuration
kubectl apply -f custom-resources.yaml
```

> 💡 Wait for all Calico pods to be Running before joining additional nodes:
> ```bash
> watch kubectl get pods -n calico-system
> ```

---

## Step 5 — Join Master Nodes (k8s-m02 and k8s-m03)

Run the **control-plane join command** from the `kubeadm init` output on each additional Master node. It looks like this:

```bash
sudo kubeadm join 192.168.73.10:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH> \
  --control-plane \
  --certificate-key <CERT_KEY>
```

Then set up kubectl on each Master node:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> ⚠️ The `--certificate-key` expires after **2 hours**. If it has expired, regenerate it on k8s-m01:
> ```bash
> kubeadm init phase upload-certs --upload-certs
> ```

---

## Step 6 — Join Worker Nodes

Run the **standard join command** from the `kubeadm init` output on each Worker node:

```bash
sudo kubeadm join 192.168.73.10:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

> 💡 Copy your actual token and hash from the `kubeadm init` output — the values above are placeholders.

---

## Verification

Check all nodes are Ready:

```bash
kubectl get nodes
```

Expected output — all nodes should show `Ready`:

```
NAME       STATUS   ROLES           AGE   VERSION
k8s-m01    Ready    control-plane   10m   v1.33.x
k8s-m02    Ready    control-plane   8m    v1.33.x
k8s-m03    Ready    control-plane   6m    v1.33.x
```

Check all system pods are Running:

```bash
kubectl get pods -A
```

> ✅ CoreDNS pods must be `Running` — they won't start until Calico is fully up. If they're stuck in `Pending`, wait a minute and check again.

---

## Notes

- The `--control-plane-endpoint` must point to the **VIP** (`192.168.73.10:6443`), not a specific Master IP. This is what makes the cluster truly HA.
- All commands in Steps 1 and 2 must be run on **every node** before initializing the cluster.
- Worker node setup is covered in the next episode.

---

*Part of the "Production-Grade Kubernetes from Scratch" series — [YouTube Channel](https://youtube.com)*
