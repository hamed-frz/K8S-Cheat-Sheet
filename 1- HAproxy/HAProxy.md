## Configuring HAProxy for Kubernetes and ETCD

### HAProxy for Kubernetes API Server

In a highly available Kubernetes cluster, HAProxy can distribute traffic across multiple Kubernetes control plane nodes, ensuring that the API remains accessible even if one or more nodes fail.


**Example HAProxy Configuration for Kubernetes API**:

```bash
frontend kubernetes_api_front
    bind *:6443
    mode tcp
    option tcplog
    default_backend kubernetes_api_back

backend kubernetes_api_back
    mode tcp
    balance roundrobin
    option tcp-check
    server k8s-master1 192.168.100.101:6443 check fall 3 rise 2
    server k8s-master2 192.168.100.102:6443 check fall 3 rise 2
    server k8s-master3 192.168.100.103:6443 check fall 3 rise 2
```

### HAProxy for ETCD

To ensure high availability and fault tolerance in ETCD, HAProxy can distribute requests to healthy ETCD nodes. This setup ensures that ETCD remains operational even if one node fails.

**Example HAProxy Configuration for ETCD**:

```bash
frontend etcd_front
    bind *:2379
    mode tcp
    option tcplog
    default_backend etcd_back

backend etcd_back
    mode tcp
    balance roundrobin
    option tcp-check
    server etcd1 192.168.100.101:2379 check fall 3 rise 2
    server etcd2 192.168.100.102:2379 check fall 3 rise 2
    server etcd3 192.168.100.103:2379 check fall 3 rise 2
```


