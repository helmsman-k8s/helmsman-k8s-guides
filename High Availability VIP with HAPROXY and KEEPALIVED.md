# High Availability Configuration with HAProxy and Keepalived

This guide explains how to configure a **Virtual IP (VIP)** using Keepalived and an HAProxy load balancer to expose the Kubernetes API Server in a resilient, highly available way.

---

## Architecture Overview

```
          PUBLIC ACCESS / ADMINISTRATOR
                      │
                      ▼
             VIRTUAL IP (VIP)
              192.168.73.10
                      │
         ┌────────────┴────────────┐
         ▼                         ▼
  HA-PROXY-01 (MASTER)      HA-PROXY-02 (BACKUP)
   192.168.73.11  ◄──VRRP──► 192.168.73.12
         │
         ▼
  KUBERNETES CONTROL PLANE
  ┌──────────────────────────────────┐
  │  k8s-m01        k8s-m02        k8s-m03  │
  │ 192.168.73.21  192.168.73.22  192.168.73.23 │
  └──────────────────────────────────┘
         │
         ▼
  KUBERNETES WORKER NODES
  ┌────────────────────────┐
  │  k8s-w01      k8s-w02  │
  │ 192.168.73.31  192.168.73.32 │
  └────────────────────────┘
```

---

## Step 1 — Installation (Both Nodes)

Update the repositories and install the required packages on **both** load balancer nodes:

```bash
sudo apt update && sudo apt install -y haproxy keepalived
```

### Enable Non-Local IP Binding

Allow HAProxy to listen on the Virtual IP even when it is not yet physically assigned to the network interface:

```bash
echo "net.ipv4.ip_nonlocal_bind=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

> ⚠️ Run this on **both nodes**. Skipping it will prevent HAProxy from starting.

---

## Step 2 — Keepalived Configuration

Keepalived manages the VIP handover between nodes. If the Master fails, the Backup takes over automatically.

### Master Node (HA-PROXY01)

Edit `/etc/keepalived/keepalived.conf`:

```
vrrp_script check_haproxy {
    script "/usr/bin/curl -f -s --max-time 3 http://127.0.0.1:10242/healthz"
    interval 3          # Run check every 3 seconds
    rise 2              # 2 positive checks = UP
    fall 3              # 3 negative checks = DOWN
    weight 2            # Increase priority if check passes
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33     # <--- Verify your interface name (e.g. eth0, ens18)
    virtual_router_id 51
    priority 101        # Higher priority = Master
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass HA-PROXY-2026
    }

    virtual_ipaddress {
        192.168.73.10/24  # Your Virtual IP (VIP)
    }

    track_script {
        check_haproxy
    }
}
```

> 💡 Check your network interface name with `ip a` before applying this config.

### Backup Node (HA-PROXY02)

Edit `/etc/keepalived/keepalived.conf` — change only `state` and `priority`:

```
vrrp_instance VI_1 {
    state BACKUP        # Set as Backup
    priority 100        # Lower priority than Master
    # ... rest of the config identical to Master ...
}
```

---

## Step 3 — HAProxy Configuration

Configure `/etc/haproxy/haproxy.cfg` on **both nodes**:

```
global
    log /dev/log local0 warning
    chroot /var/lib/haproxy
    maxconn 4000
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode tcp
    option tcplog
    timeout connect 5s
    timeout client 50s
    timeout server 50s

# Internal health check endpoint for Keepalived
frontend health_check
    bind 127.0.0.1:10242
    mode http
    monitor-uri /healthz

# Kubernetes API Server load balancing
frontend k8s-api
    bind 192.168.73.10:6443        # Listen on VIP port 6443
    mode tcp
    default_backend kube-apiserver

backend kube-apiserver
    mode tcp
    balance roundrobin
    option tcp-check
    # Replace with your actual Master node IPs
    server k8s-m01 192.168.73.21:6443 check
    server k8s-m02 192.168.73.22:6443 check
    server k8s-m03 192.168.73.23:6443 check
```

> 💡 Replace the backend server IPs with the actual IPs of your Kubernetes Master nodes.

---

## Step 4 — Restart Services

Apply the configuration by restarting both services on **both nodes**:

```bash
sudo systemctl restart haproxy
sudo systemctl restart keepalived
```

---

## Step 5 — Firewall Configuration

Keepalived uses the **VRRP protocol** to communicate between nodes. You must allow it explicitly, otherwise failover will silently break.

### UFW

```bash
sudo ufw allow proto vrrp
sudo ufw allow 6443/tcp
```

### iptables (if UFW gives issues)

```bash
sudo iptables -I INPUT -p 112 -j ACCEPT
sudo iptables -I INPUT -p tcp --dport 6443 -j ACCEPT
sudo apt install -y iptables-persistent && sudo netfilter-persistent save
```

---

## Step 6 — Verification

### Check VIP Assignment

On the Master node, the VIP should appear:

```bash
ip addr show | grep 192.168.73.10
```

### Failover Test

Stop Keepalived on the **Master**:

```bash
sudo systemctl stop keepalived
```

Check that the VIP has moved to the **Backup**:

```bash
ip addr show | grep 192.168.73.10
```

### API Server Test

Contact the cluster through the VIP:

```bash
curl -k https://192.168.73.10:6443/healthz
```

> ✅ Expected result: `401 Unauthorized` — the server is responding correctly, it just requires credentials. This confirms everything is working.

---

## Notes

- This setup uses a **production-grade architecture** pattern (VIP + dual load balancers + multi-master) running on a home lab environment.
- The `virtual_router_id` must be **unique** on your network segment. If you have multiple VRRP instances, use different IDs.
- The `auth_pass` value in Keepalived is shared between Master and Backup — make sure they match exactly.

---

*Part of the "Production-Grade Kubernetes from Scratch" series — [YouTube Channel](https://youtube.com)*
