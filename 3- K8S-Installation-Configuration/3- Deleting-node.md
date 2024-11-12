## Deleting Worker Nodes from the K8S Cluster

If you need to remove a worker node from your Kubernetes cluster, follow these steps to do so safely and cleanly. This ensures that the node is properly drained, all resources are cleared, and the node is removed from the cluster.

### Steps to Delete a Worker Node

1. **List all nodes:**

    First, get a list of all nodes in your Kubernetes cluster to identify the node you want to delete.

    ```bash
    kubectl get nodes
    ```

2. **Drain the node:**

    Draining a node safely evicts all the pods running on the node, except for those managed by DaemonSets. This step is crucial to ensure workloads are not abruptly interrupted.

    ```bash
    kubectl drain <node-name> --ignore-daemonsets --delete-local-data
    ```

    Replace `<node-name>` with the name of the node you want to delete. The `--ignore-daemonsets` flag allows the drain process to ignore DaemonSet-managed pods, and `--delete-local-data` ensures that any local data on the node is deleted.

3. **Delete the node from the cluster:**

    After draining the node, remove it from the cluster with the following command:

    ```bash
    kubectl delete node <node-name>
    ```

    Replace `<node-name>` with the name of the node you just drained.

4. **Reset the node (on the worker node itself):**

    Finally, if you want to completely remove Kubernetes configuration from the node, reset the node using `kubeadm`. This command removes all Kubernetes-related configurations, making the node clean.

    ```bash
    kubeadm reset
    ```

    Run this command on the worker node itself after it has been deleted from the cluster.