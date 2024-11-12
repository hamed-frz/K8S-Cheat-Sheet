## Validation

1. **Check Node Status**: Ensure that all nodes are in `Ready` status.

    ```bash
    kubectl get nodes
    ```

2. **Verify Cluster Components**: Check the health of core components in the cluster.

    ```bash
    kubectl get cs
    ```

3. **Inspect Logs**: Review `kubelet` logs to confirm there are no errors post-upgrade.

    ```bash
    sudo journalctl -xeu kubelet
    ```
