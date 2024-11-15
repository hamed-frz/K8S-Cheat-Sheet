## Kubernetes Cluster Deployment

### Enable iptables Bridged Traffic on All Nodes

To ensure that network traffic is properly routed through iptables, enable bridged traffic on all nodes (both master and worker nodes) with the following commands:

```bash
sudo apt update && sudo apt install -y net-tools

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

```

### Configure sysctl Parameters for Kubernetes

The following sysctl parameters are required for the Kubernetes setup. These parameters ensure proper network packet forwarding and that iptables can correctly handle bridged network traffic. These settings will persist across reboots.

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

```bash
sudo sysctl --system
```

### Install CRI-O Runtime on All Nodes

CRI-O is a lightweight container runtime specifically designed for Kubernetes. It is an alternative to Docker and is optimized for use with Kubernetes. Follow these steps to install CRI-O on all nodes in your Kubernetes cluster.

1. **Update the package list and install prerequisites:**

    First, ensure that your package list is up-to-date and install the necessary packages to add the CRI-O repository.

    ```bash
    sudo apt-get update -y
    sudo apt-get install -y software-properties-common curl apt-transport-https ca-certificates
    ```

2. **Add the CRI-O repository:**

    Add the CRI-O repository to your system. This step involves downloading the repository's GPG key and adding the repository to your system's sources list.

    ```bash
    curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key | \
        gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
    echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" | \
        sudo tee /etc/apt/sources.list.d/cri-o.list
    ```

3. **Install CRI-O:**

    Update your package list again to include the newly added CRI-O repository, then install the CRI-O runtime.

    ```bash
    sudo apt-get update -y
    sudo apt-get install -y cri-o
    ```

4. **Start and enable the CRI-O service:**

    After installation, ensure that the CRI-O service is started and enabled to start automatically at boot.

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable crio --now
    sudo systemctl start crio.service
    ```

5. **Install `crictl` command-line tool:**

    `crictl` is a command-line tool for interacting with CRI-O and other container runtimes. Download and install `crictl` on all nodes.

    ```bash
    VERSION="v1.30.1"
    wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
    sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
    rm -f crictl-$VERSION-linux-amd64.tar.gz
    ```

At this point, CRI-O is installed and configured on all your nodes, and the `crictl` tool is available for managing containers.


### Install Kubeadm & Kubelet & Kubectl on all Nodes

Set the Kubernetes version and prepare the system by setting up the necessary keyrings and repositories.

```bash
KUBERNETES_VERSION=1.30
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v$KUBERNETES_VERSION/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update the package list and install the necessary Kubernetes tools.

```bash
sudo apt-get update -y
apt-cache madison kubeadm | tac
sudo apt-get install -y kubelet kubeadm kubectl
# Alternatively, specify the version:
# sudo apt-get install -y kubelet=1.30.3-1.1 kubectl=1.30.3-1.1 kubeadm=1.30.3-1.1
```

Hold the versions to prevent unintended upgrades.

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Install `jq` for JSON processing, then set up the Kubelet configuration to use the node’s IP address.

```bash
NIC="ens160"

sudo apt-get install -y jq
local_ip="$(ip --json addr show $NIC | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=$local_ip
EOF
```


## Initializing the Master Node to K8S Cluster

After setting up and configuring ETCD, the next step is to initialize the master node(s) in your Kubernetes cluster. This involves configuring Kubernetes to use the external ETCD cluster and setting up the necessary network configurations.

### Steps to Initialize the Master Node

1. **Prepare the directory for ETCD certificates:**

    Create the required directory on the master node and copy the ETCD certificates to it.

    ```bash
    mkdir -p /etc/kubernetes/pki/etcd
    cp ca.pem etcd.pem etcd-key.pem /etc/kubernetes/pki/etcd/
    ```

2. **Set the IP addresses:**

    Define the IP addresses for the ETCD nodes and the high availability (HA) IP for the control plane. Replace these with your actual IPs.

    ```bash
    // Set the node IPs (replace these with the actual IPs)
    ETCD1_IP="192.168.100.101"
    ETCD2_IP="192.168.100.102"
    ETCD3_IP="192.168.100.103"
    HA_IP="192.168.100.100"
    ```

3. **Create the `kubeadm` configuration file:**

    This configuration file specifies how Kubernetes should initialize the cluster, including the use of external ETCD nodes and network settings.

    ```bash
    cat <<EOF > kubeadm-config.yaml
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: ClusterConfiguration
    controlPlaneEndpoint: ${HA_IP}:6443
    etcd:
        external:
            endpoints:
            - https://${ETCD1_IP}:2379
            - https://${ETCD2_IP}:2379
            - https://${ETCD3_IP}:2379
            caFile: /etc/kubernetes/pki/etcd/ca.pem
            certFile: /etc/kubernetes/pki/etcd/etcd.pem
            keyFile: /etc/kubernetes/pki/etcd/etcd-key.pem
    networking:
      dnsDomain: cluster.local
      podSubnet: "172.27.0.0/16"
      serviceSubnet: 10.96.0.0/12
    apiServer:
      certSANs:
        - ${HA_IP}
      extraArgs:
        apiserver-count: "3"
    EOF
    ```

    This file sets up the control plane to use the external ETCD cluster and defines the networking settings for the Kubernetes cluster.

4. **Initialize the master node:**

    Use `kubeadm` to initialize the Kubernetes cluster with the configuration file you created.

    ```bash
    sudo kubeadm init --config=kubeadm-config.yaml --upload-certs
    ```

    This command initializes the master node, setting up the control plane and connecting it to the ETCD cluster.




