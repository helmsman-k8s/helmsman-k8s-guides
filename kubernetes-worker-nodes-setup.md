# Kubernetes Worker Nodes Setup (v1.33)

**Worker Nodes:** `k8s-w01` → `192.168.73.31` | `k8s-w02` → `192.168.73.32` | **VIP:** `192.168.73.10`

> ⚠️ Before starting this guide, make sure Episodes 1 and 2 are complete — the VIP must be up and the Control Plane must be fully initialized.

---

## Step 1 — Preparation (Both Worker Nodes)

Run this on **both worker nodes**.

```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/^\s*[^#]*swap\s/ s/^/#/' /etc/fstab

# Configure /etc/hosts
echo -e "192.168.73.21 k8s-m01\n192.168.73.22 k8s-m02\n192.168.73.23 k8s-m03\n192.168.73.10 k8s-vip\n192.168.73.31 k8s-w01\n192.168.73.32 k8s-w02" | sudo tee -a /etc/hosts > /dev/null

# Reboot
sudo reboot
```

---

## Step 2 — Install Containerd and Kubernetes v1.33 (Both Worker Nodes)

Identical to the Master node setup — run on **both workers**.

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

```bash
# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay && sudo modprobe br_netfilter

# Configure sysctl
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

> ⚠️ `SystemdCgroup = true` is required. Don't skip this step.

---

## Step 3 — Join the Cluster (Both Worker Nodes)

Run the worker join command from your `kubeadm init` output on **both worker nodes**:

```bash
sudo kubeadm join 192.168.73.10:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

> 💡 If your token has expired (they expire after 24 hours), regenerate it from any Master node:
> ```bash
> kubeadm token create --print-join-command
> ```
> This prints a fresh, ready-to-use join command.

---

## Step 4 — Verify and Label the Nodes (From any Master node)

Check all nodes are visible and Ready:

```bash
kubectl get nodes
```

Expected output:

```
NAME       STATUS   ROLES           AGE   VERSION
k8s-m01    Ready    control-plane   30m   v1.33.x
k8s-m02    Ready    control-plane   28m   v1.33.x
k8s-m03    Ready    control-plane   26m   v1.33.x
k8s-w01    Ready    <none>          2m    v1.33.x
k8s-w02    Ready    <none>          1m    v1.33.x
```

> 💡 Notice the worker nodes show `<none>` as their role. Let's fix that by adding the `worker` label:

```bash
kubectl label node k8s-w01 node-role.kubernetes.io/worker=worker
kubectl label node k8s-w02 node-role.kubernetes.io/worker=worker
```

Now verify:

```bash
kubectl get nodes
```

Expected output:

```
NAME       STATUS   ROLES           AGE   VERSION
k8s-m01    Ready    control-plane   30m   v1.33.x
k8s-m02    Ready    control-plane   28m   v1.33.x
k8s-m03    Ready    control-plane   26m   v1.33.x
k8s-w01    Ready    worker          2m    v1.33.x
k8s-w02    Ready    worker          1m    v1.33.x
```

---

## Step 5 — Smoke Test (Deploy a Test Pod)

Let's verify the cluster is actually scheduling workloads on the worker nodes:

```bash
# Deploy a simple nginx pod
kubectl run nginx-test --image=nginx --restart=Never

# Check it's running and note which node it landed on
kubectl get pod nginx-test -o wide
```

Expected output — the pod should be on one of the worker nodes:

```
NAME         READY   STATUS    NODE
nginx-test   1/1     Running   k8s-w01
```

Clean up:

```bash
kubectl delete pod nginx-test
```

> ✅ If the pod is Running on a worker node — your cluster is complete and fully operational.

---

## Full Cluster Overview

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

All nodes `Ready`, all system pods `Running`. That's your production-grade bare metal Kubernetes cluster — fully operational.

---

## Notes

- Worker nodes only run the **kubelet**, **kube-proxy**, and your workloads — they do not run API server, etcd, or scheduler.
- If a pod lands on a Master node, it means the control-plane taint was removed. You can re-apply it with:
  ```bash
  kubectl taint node k8s-m01 node-role.kubernetes.io/control-plane:NoSchedule
  kubectl taint node k8s-m02 node-role.kubernetes.io/control-plane:NoSchedule
  kubectl taint node k8s-m03 node-role.kubernetes.io/control-plane:NoSchedule
  ```
- In the next episode we install **Rancher** for cluster management via UI.

---

*Part of the "Production-Grade Kubernetes from Scratch" series — [YouTube Channel](https://youtube.com)*
