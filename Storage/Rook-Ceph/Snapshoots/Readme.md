# Snapshots

## RBD Snapshots
### Take snapshot
1. **RBD VolumeSnapshotClass:**

    ```bash
    kubectl create -f deploy/examples/csi/rbd/snapshotclass.yaml
    ```

2. **RBD Volumesnapshot:**

    ```bash
    kubectl create -f deploy/examples/csi/rbd/snapshot.yaml
    ```

3. **RBD Volumesnapshot:**
    ```bash
    kubectl get volumesnapshotclass
    kubectl get volumesnapshot
    ```

---
### Snapshot resource
1. **Restore the RBD snapshot to a new PVC:**
    ```bash
    kubectl create -f deploy/examples/csi/rbd/pvc-restore.yaml
    ```

2. **Verify RBD Clone PVC Creation:**
    ```bash
    kubectl get pvc
    ```

3. **RBD snapshot resource Cleanup:**
    ```bash
    kubectl delete -f deploy/examples/csi/rbd/pvc-restore.yaml
    kubectl delete -f deploy/examples/csi/rbd/snapshot.yaml
    kubectl delete -f deploy/examples/csi/rbd/snapshotclass.yaml
    ```

