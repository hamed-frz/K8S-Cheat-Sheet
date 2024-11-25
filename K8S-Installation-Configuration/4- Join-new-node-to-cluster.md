## Joining Additional Master Nodes to the K8S Cluster

After initializing the first master node in your Kubernetes cluster, you may want to join additional master nodes to create a highly available control plane. This section provides the steps to join additional master nodes to the cluster.

### Joining Worker Nodes to the K8S Cluster

1. **Create a token to join the node:**

    On the master node, create a token that will be used by the worker nodes to join the cluster.

    ```bash
    kubeadm token create --print-join-command
    ```

    This command will generate a `kubeadm join` command with the appropriate token and CA cert hash, which you will run on the worker node.

2. **Run the join command on the worker node:**

    Execute the generated `kubeadm join` command on the worker node to join it to the cluster. Replace `<change it>` with the token and CA cert hash provided by the `kubeadm token create` command.

    ```bash
    kubeadm join 192.168.100.100:6443 --token <change it> --discovery-token-ca-cert-hash <change it>
    ```

    This command connects the worker node to the Kubernetes cluster, enabling it to start running workloads.

3. **Label the node (optional):**

    If you want to label the worker node with a specific role, you can do so with the following command:

    ```bash
    kubectl label node node01 node-role.kubernetes.io/worker=worker
    ```

    This label helps identify the node's role within the cluster.


### Joining Master Nodes to the K8S Cluster

1. **Generate a new certificate key (if necessary):**

    If you need to generate a new certificate key for the additional master node, use the following command:

    ```bash
    kubeadm init phase upload-certs --upload-certs --config=kubeadm-config.yaml
    ```

2. **Join the master node to the cluster:**

    Use the `kubeadm join` command to join the master node to the existing cluster. Replace `<changeit>` with the appropriate token, CA cert hash, and certificate key.

    ```bash
    kubeadm join 192.168.100.100:6443 --token <changeit> \
            --discovery-token-ca-cert-hash <changeit> \
            --control-plane --certificate-key <changeit>
    ```

    This command connects the new master node to the existing cluster, ensuring it becomes part of the control plane.

3. **Allow pods to run on the master node (optional):**

    If you want to allow pods to run on the master node, you can remove the taint that prevents workloads from being scheduled on the control plane nodes:

    ```bash
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-
    ```

    The command should return the following message:

    ```bash
    node/<your-hostname> untainted
    ```


### Additional Operations

- **Rejoin with another IP (if needed):**

    If you need to rejoin the worker node with a different IP address, update the following file:

    ```bash
    /etc/default/kubelet:
      KUBELET_EXTRA_ARGS=--node-ip=192.168.100.104
    ```

    After updating the IP, restart the `kubelet` service to apply the changes.

    ```bash
    systemctl restart kubelet
    ```
