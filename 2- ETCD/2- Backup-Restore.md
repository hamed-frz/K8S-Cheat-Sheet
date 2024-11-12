### Consider Using an External Backup and Restore Solution

Backup a Snapshot:
For production environments, you should automate ETCD snapshots and back them up to a remote storage solution.

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/pki/ca.pem \
  --cert=/etc/etcd/pki/etcd.pem \
  --key=/etc/etcd/pki/etcd-key.pem \
  snapshot save /backup/etcd-snapshot-$(date +%Y%m%d-%H%M%S).db
```

Restore a Snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db --data-dir=/var/lib/etcd
```

The etcd-kube-control-plane Pod will be re-created, and it points to the restored backup directory
In case the Pod doesn’t transition into the “Running” status, try to delete it manually with the command:

```bash
kubectl delete pod etcd-kube-control-plane -n kube-system
```