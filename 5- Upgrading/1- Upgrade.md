
# Kubernetes Cluster Upgrade Guide

This guide provides a step-by-step process to upgrade both control plane and worker nodes in a Kubernetes cluster
---

## Prerequisites

1. Verify you have administrative access to all nodes in the Kubernetes cluster.
2. Backup etcd data and all Kubernetes configuration files.
3. Confirm your current Kubernetes version supports direct upgrades to the target version.
4. Notify stakeholders about potential downtime during the upgrade process.

---

## Control Plane Node Upgrade

### Step 1: Drain the Control Plane Node

To begin, drain the control plane node to safely evict workloads.

```bash
kubectl drain kube-control-plane --ignore-daemonsets --delete-local-data
```

### Step 2: Check Current Kubeadm Version

Verify the current version of Kubeadm.

```bash
sudo apt-cache madison kubeadm
```

### Step 3: Upgrade Kubeadm

Upgrade Kubeadm to the target version. Replace `<version>` with the desired version number.

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=<version> && sudo apt-mark hold kubeadm
kubeadm version
```

### Step 4: Plan the Kubeadm Upgrade

Run the upgrade plan to review available updates and changes.

```bash
sudo kubeadm upgrade plan
```

**Note**: Before applying the upgrade, ensure that `kubeadm` is updated to match the target version.

### Step 5: Apply the Kubeadm Upgrade

Execute the upgrade with the following command:

```bash
sudo kubeadm upgrade apply <version>
```

### Step 6: Upgrade Kubelet and Kubectl

Upgrade the Kubelet and Kubectl tools to the target version.

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=<version> kubectl=<version> && sudo apt-mark hold kubelet kubectl
```

### Step 7: Restart the Kubelet

After upgrading Kubelet, restart its service.

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Step 8: Uncordon the Control Plane Node

Once the upgrade is complete, make the node schedulable again.

```bash
kubectl uncordon kube-control-plane
```

---

## Worker Node Upgrade

### Step 1: Upgrade Kubeadm on Worker Node

Start by upgrading `kubeadm` on the worker node. Replace `<version>` with the target version.

```bash
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install -y kubeadm=<version> && sudo apt-mark hold kubeadm
kubeadm version
```

### Step 2: Upgrade Kubelet Configuration

Upgrade the Kubelet configuration on the worker node.

```bash
sudo kubeadm upgrade node
```

### Step 3: Drain the Worker Node

Evict workloads from the worker node to prevent scheduling during the upgrade.

```bash
kubectl drain kube-worker-1 --ignore-daemonsets --delete-local-data
```

### Step 4: Upgrade Kubelet and Kubectl on Worker Node

Upgrade Kubelet and Kubectl to the target version for the worker node.

```bash
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get install -y kubelet=<version> kubectl=<version> && sudo apt-mark hold kubelet kubectl
```

### Step 5: Restart the Kubelet Service

Reload and restart the Kubelet service on the worker node.

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Step 6: Uncordon the Worker Node

Make the worker node schedulable again after the upgrade is completed.

```bash
kubectl uncordon kube-worker-1
```

## Rollback (If Needed)

If issues arise during the upgrade, follow these steps to rollback:
1. Downgrade `kubeadm`, `kubelet`, and `kubectl` to the previous stable version.
2. Restore the etcd data and configuration files from the backup.

---

This guide provides the necessary steps to upgrade both control plane and worker nodes in a Kubernetes cluster while ensuring minimum downtime.