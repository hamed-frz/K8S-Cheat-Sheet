### Troubleshooting ETCD

1. **Check ETCD Cluster Health**

    The first step in verifying that your ETCD configuration is working correctly is to check the overall health of the cluster. You can use the etcdctl command-line tool to do this.

    ```bash
    ETCDCTL_API=3 etcdctl \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/etcd/pki/ca.pem \
        --cert=/etc/etcd/pki/etcd.pem \
        --key=/etc/etcd/pki/etcd-key.pem \
        endpoint health
    ```

2. **Verify that ETCD is running and the nodes are correctly configured:**

    Use `etcdctl` to check the status of the ETCD cluster and ensure that all nodes are up and running.

    ```bash
    ETCDCTL_API=3 etcdctl \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/pki/ca.pem \
      --cert=/etc/etcd/pki/etcd.pem \
      --key=/etc/etcd/pki/etcd-key.pem \
      member list
    ```

    This command will list all members of the ETCD cluster along with their statuses.
3. **Check ETCD Leader Election and Raft Status**

    To verify the health of the leader election and the consistency of the Raft protocol, you can use the etcdctl endpoint status command.

    ```bash
    ETCDCTL_API=3 etcdctl \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/pki/ca.pem \
      --cert=/etc/etcd/pki/etcd.pem \
      --key=/etc/etcd/pki/etcd-key.pem \
      endpoint status --write-out=table
    ```
    Leader: Only one node should have isLeader=true. Frequent changes in leadership can indicate instability in the cluster.
