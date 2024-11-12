## K8S Cluster Cheat Sheet

This documentation serves as a comprehensive cheat sheet for setting up a Highly Available (HA) Kubernetes Cluster.

### The cluster inforamtion:

- Provisioned the Kubernetes cluster using kubeadm.
- Configured an external Etcd cluster.
- Implemented HAProxy to aid with scalability and redundancy.
- Need at least 3 master nodes and 3 worker nodes.
- Utilized Rook-Ceph for storage management within the Kubernetes cluster.


### The cluster Enviroment
|Role|FQDN|IP|OS|
|----|----|----|----|
|etcd1|etcd1.k8s.local|192.168.100.101|Ubuntu 22.04|
|etcd2|etcd2.k8s.local|192.168.100.102|Ubuntu 22.04|
|etcd3|etcd3.k8s.local|192.168.100.103|Ubuntu 22.04|
|HAProxy|haproxy.k8s.local|192.168.100.100|ubuntu 22.04|
|Master1|master-node1.k8s.local|192.168.100.101|Ubuntu 22.04|
|Master2|master-node2.k8s.local|192.168.100.102|Ubuntu 22.04|
|Master3|master-node3.k8s.local|192.168.100.103|Ubuntu 22.04|
|Worker1|worker-node1.k8s.local|192.168.100.104|Ubuntu 22.04|
|Worker2|worker-node2.k8s.local|192.168.100.105|Ubuntu 22.04|
|Worker3|worker-node3.k8s.local|192.168.100.106|Ubuntu 22.04|
