# Upgrading

### The examples given in this guide upgrade a live Rook cluster

**Parameterize the environment:**
```bash
export ROOK_OPERATOR_NAMESPACE=rook-ceph
export ROOK_CLUSTER_NAMESPACE=rook-ceph
export VERSION=v1.15.6
```

1. **Update common resources and CRDs:**

    - ***Get the latest common resources***
        ```bash
        git clone --single-branch --depth=1 --branch v1.14.12 https://github.com/rook/rook.git
        cd rook/deploy/examples
        ```

    - ***If the Rook Operator or CephCluster are deployed into a different namespace:***
        ```bash
        sed -i.bak \
            -e "s/\(.*\):.*# namespace:operator/\1: $ROOK_OPERATOR_NAMESPACE # namespace:operator/g" \
            -e "s/\(.*\):.*# namespace:cluster/\1: $ROOK_CLUSTER_NAMESPACE # namespace:cluster/g" \
        common.yaml
        ```

    - ***Apply the resources:***
        ```bash
        kubectl apply -f common.yaml -f crds.yaml
        ```


2. **Update the Rook Operator:**

    - ***The largest portion of the upgrade is triggered when the operator's image is updated***
        ```bash
        kubectl -n $ROOK_OPERATOR_NAMESPACE set image deploy/rook-ceph-operator rook-ceph-operator=rook/ceph:v1.14.12
        ```

3. **Update Ceph CSI:**

    ***Note:***
        This is automatically updated if custom CSI image versions are not set.



4. **viewed as they are updated:**
    ```bash
    watch --exec kubectl -n $ROOK_CLUSTER_NAMESPACE get deployments -l rook_cluster=$ROOK_CLUSTER_NAMESPACE -o jsonpath='{range .items[*]}{.metadata.name}{"  \treq/upd/avl: "}{.spec.replicas}{"/"}{.status.updatedReplicas}{"/"}{.status.readyReplicas}{"  \trook-version="}{.metadata.labels.rook-version}{"\n"}{end}'
    ```