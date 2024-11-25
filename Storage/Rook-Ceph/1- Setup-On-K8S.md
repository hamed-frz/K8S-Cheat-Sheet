# Rook-Ceph Installation and Configuration Cheet-Sheat

## Prerequisites
1. **Kubernetes Cluster**: Ensure you have a properly configured and running Kubernetes cluster with sufficient resources for Rook-Ceph.
2. **kubectl**: Make sure `kubectl` is installed and correctly configured to communicate with your Kubernetes cluster.
3. **Node Preparation**: Identify nodes that will be dedicated to storage and label them accordingly.

---

## Step 1: Clone the Rook Repository
Get the Rook deployment examples by cloning the Rook repository:

```bash
git clone --single-branch --branch v1.14.9 https://github.com/rook/rook.git

```
---

## Step 2: Prepare Nodes for Rook-Ceph (Optional)
If Rook-Ceph needs to run on specific nodes, taint those nodes to restrict scheduling:

```bash
kubectl taint nodes worker-node1 role=storage-node:NoSchedule
kubectl taint nodes worker-node2 role=storage-node:NoSchedule
kubectl taint nodes worker-node3 role=storage-node:NoSchedule
```

### Update `rook/deploy/examples/operator.yaml` with Tolerations
You must configure `rook/deploy/examples/operator.yaml` to handle node taints. Modify the `tolerations` section as follows:

```yaml
tolerations:
  - key: "role"
    operator: "Equal"
    value: "storage-node"
    effect: "NoSchedule"
```

**Note**:  
- We use node taints and tolerations to ensure that Rook-Ceph pods are scheduled only on nodes designated for storage. This setup isolates storage operations from other workloads, improving performance and reliability. The role=storage-node taint and corresponding toleration in operator.yaml are critical for enforcing this scheduling policy. Do not modify these settings without understanding the potential impact on the cluster's storage architecture.


---

## Step 3: Configure Rook-Ceph Operator
1. **Automatic Device Detection (Optional)**:  
   If you want Rook-Ceph to automatically discover raw storage devices, enable this in `rook/deploy/examples/operator.yaml`:

   ```yaml
   useAllDevices: true
   ```

2. **Custom Configuration**:  
   - Ensure you customize settings such as resource requests/limits and storage settings according to your environment's needs.
   - Consider adding detailed documentation in the `rook/deploy/examples/operator.yaml` to describe any custom configurations.

---

## Step 4: Deploy Rook-Ceph
Apply the necessary manifests in the following order:

```bash
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml
kubectl create -f operator.yaml
kubectl create -f cluster.yaml
```

---

## Step 5: Deploy Rook-Ceph Toolbox (Optional)
The Rook-Ceph toolbox contains tools that are useful for debugging and managing your Ceph cluster.

```bash
kubectl create -f toolbox.yaml
kubectl exec -n rook-ceph -it $(kubectl get po -n rook-ceph | egrep rook-ceph-tools | awk '{print $1}') -- /bin/bash
```

Check the Ceph status:

```bash
ceph status
```

**Recommendation**:  
- Use a secure method to restrict access to the toolbox in production environments.

---

## Step 6: Create a StorageClass
Deploy the Rook-managed Ceph StorageClass for block storage:

```bash
kubectl create -f storageclass.yaml
```

### Verify the StorageClass
Ensure the `StorageClass` is created and properly configured:

```bash
kubectl get storageclass
```

**Recommendation**:  
- Customize `storageclass.yaml` as needed for your storage requirements, such as setting the appropriate replication factor or pool settings.

---

## Step 7: Access the Rook-Ceph Dashboard
Retrieve the admin credentials to access the Rook-Ceph dashboard:

- **Username**: `admin`
- **Password**: Run the following command:

  ```bash
  kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o yaml | grep "password:" | awk '{print $2}' | base64 --decode
  ```

**Dashboard URL**: Refer to the documentation for instructions on accessing the dashboard.

**Recommendation**:  
- Secure the dashboard access using network policies or a reverse proxy to restrict unauthorized access.

---

## Additional Resources
- **Rook Quickstart Guide**: [Getting Started with Rook](https://rook.io/docs/rook/latest-release/Getting-Started/quickstart/)
- **Block Storage Configuration**: [Block Storage Setup](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/block-storage/#provision-storage)
- **Useful Articles**:
  - [Deploying a Ceph Cluster with Kubernetes and Rook](https://dev.to/itminds/deploying-a-ceph-cluster-with-kubernetes-and-rook-1291)
  - [Ceph Data Durability and Redundancy](https://dev.to/itminds/ceph-data-durability-redundancy-and-how-to-use-ceph-2ml0)
  - [Rook as a Storage Orchestrator](https://medium.com/devopsturkiye/rook-a-storage-orchestrator-to-run-stateful-workloads-on-kubernetes-with-ceph-500882ecf005)

---

**Final Recommendations**:
- **Monitoring**: Integrate Prometheus and Grafana for monitoring the health and performance of your Ceph cluster.
- **Backups**: Regularly back up your Ceph configuration and data, especially in production environments.
- **Security**: Implement RBAC policies and secure network configurations to protect your Rook-Ceph deployment.
