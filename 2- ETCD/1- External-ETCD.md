## Configuring ETCD

ETCD is a distributed key-value store that is used as the backend for Kubernetes. Configuring ETCD correctly is crucial for the reliability and performance of your Kubernetes cluster. Below are the steps to install and configure ETCD on your cluster.

### Installing ETCD

1. **Create a directory for ETCD and navigate into it:**

    Begin by creating a directory to hold the ETCD installation files.

    ```bash
    mkdir -p etcd && cd etcd
    ```

2. **Download and install CFSSL tools:**

    CFSSL (Cloudflare's PKI and TLS toolkit) is used to create the necessary certificates for securing ETCD.

    ```bash
    wget -q --show-progress \
        https://github.com/cloudflare/cfssl/releases/download/v1.6.5/cfssl_1.6.5_linux_amd64 \
        https://github.com/cloudflare/cfssl/releases/download/v1.6.5/cfssljson_1.6.5_linux_amd64
    chmod +x cfssl_1.6.5_linux_amd64 cfssljson_1.6.5_linux_amd64
    sudo mv cfssl_1.6.5_linux_amd64 cfssl && mv cfssljson_1.6.5_linux_amd64 cfssljson
    sudo mv cfssl cfssljson /usr/local/bin/
    ```

3. **Create the CA configuration file:**

    This file specifies the default configuration for your Certificate Authority (CA), including the expiration time and usage profiles.

    ```bash
    cat > ca-config.json <<EOF
    {
        "signing": {
            "default": {
                "expiry": "87600h"
            },
            "profiles": {
                "etcd": {
                    "expiry": "87600h",
                    "usages": ["signing","key encipherment","server auth","client auth"]
                }
            }
        }
    }
    EOF
    ```

4. **Create the CA Certificate Signing Request (CSR):**

    This JSON file defines the details for your CA certificate, such as the common name (CN), organization, and location.

    ```bash
    cat > ca-csr.json <<EOF
    {
      "CN": "etcd cluster",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "GB",
          "L": "England",
          "O": "Kubernetes",
          "OU": "ETCD-CA",
          "ST": "Cambridge"
        }
      ]
    }
    EOF
    ```

5. **Generate the CA certificate and key:**

    Use CFSSL to generate the CA certificate (`ca.pem`) and private key (`ca-key.pem`).

    ```bash
    cfssl gencert -initca ca-csr.json | cfssljson -bare ca
    ```

6. **Set the IP addresses for the ETCD nodes:**

    Define the IP addresses for the ETCD nodes in your cluster. Replace these with the actual IPs of your ETCD nodes.

    ```bash
    ETCD1_IP="192.168.100.101"
    ETCD2_IP="192.168.100.102"
    ETCD3_IP="192.168.100.103"
    HA_PROXY_IP="192.168.100.100"
    ```

7. **Create the ETCD CSR configuration file:**

    This file specifies the details for the ETCD server certificates, including the IP addresses of the ETCD nodes.

    ```bash
    cat > etcd-csr.json <<EOF
    {
      "CN": "etcd",
      "hosts": [
        "localhost",
        "127.0.0.1",
        "${ETCD1_IP}",
        "${ETCD2_IP}",
        "${ETCD3_IP}",
        "${HA_PROXY_IP}"
      ],
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "GB",
          "L": "England",
          "O": "Kubernetes",
          "OU": "etcd",
          "ST": "Cambridge"
        }
      ]
    }
    EOF
    ```

8. **Generate the ETCD server certificates and key:**

    Use CFSSL to generate the ETCD server certificate (`etcd.pem`) and key (`etcd-key.pem`).

    ```bash
    cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd
    ```

These steps set up the necessary certificates for ETCD, which are crucial for securing communication between ETCD nodes and the Kubernetes cluster.


## Configuring ETCD

ETCD is a distributed key-value store that is crucial for Kubernetes, as it stores all cluster data. This section covers the setup of ETCD on multiple nodes, ensuring that it is properly configured for high availability and security.

### Distribute ETCD Certificates to All Nodes

After generating the ETCD certificates, you need to distribute them to all ETCD nodes in your cluster.

1. **Copy the certificates to all nodes:**

    Define the IP addresses of your ETCD nodes and use `scp` to copy the necessary certificates to each node.

    ```bash
    // Define the node IP addresses
    declare -a NODES=(192.168.100.101 192.168.100.102 192.168.100.103)

    // Copy the certificates to each node
    for node in ${NODES[@]}; do
      scp ca.pem etcd.pem etcd-key.pem root@$node:/root/
    done
    ```

2. **Apply the certificates on all nodes:**

    On each node, create the required directory and move the certificates into place.

    ```bash
    mkdir -p /etc/etcd/pki
    cp ca.pem etcd.pem etcd-key.pem /etc/etcd/pki/
    ```

### Install ETCD on All Nodes

1. **Download and install ETCD:**

    Download the ETCD binary and install it on all nodes.

    ```bash
    ETCD_VER=v3.5.15
    wget -q --show-progress "https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz"
    tar zxf etcd-v3.5.15-linux-amd64.tar.gz
    mv etcd-v3.5.15-linux-amd64/etcd* /usr/local/bin/
    rm -rf etcd*
    ```

### Configure the ETCD Service

1. **Set up the ETCD service:**

    Define the necessary environment variables and create the systemd service file for ETCD. Ensure that the `NODE_IP` variable is set to the IP address of the current node.

    ```bash
    // Set the node IP (replace this with the actual IP of the node)
    NODE_IP="192.168.100.101"

    ETCD_NAME=$(hostname -s)

    ETCD1_IP="192.168.100.101"
    ETCD2_IP="192.168.100.102"
    ETCD3_IP="192.168.100.103"
    HA_PROXY_IP="192.168.100.100"

    // Create the ETCD systemd service file
    cat <<EOF >/etc/systemd/system/etcd.service
    [Unit]
    Description=etcd

    [Service]
    Type=notify
    ExecStart=/usr/local/bin/etcd \\
      --metrics=extensive
      --name ${ETCD_NAME} \\
      --cert-file=/etc/etcd/pki/etcd.pem \\
      --key-file=/etc/etcd/pki/etcd-key.pem \\
      --peer-cert-file=/etc/etcd/pki/etcd.pem \\
      --peer-key-file=/etc/etcd/pki/etcd-key.pem \\
      --trusted-ca-file=/etc/etcd/pki/ca.pem \\
      --peer-trusted-ca-file=/etc/etcd/pki/ca.pem \\
      --peer-client-cert-auth \\
      --client-cert-auth \\
      --initial-advertise-peer-urls https://${NODE_IP}:2380 \\
      --listen-peer-urls https://${NODE_IP}:2380 \\
      --advertise-client-urls https://${HA_PROXY_IP}:2379 \\
      --listen-client-urls https://${NODE_IP}:2379,https://127.0.0.1:2379 \\
      --initial-cluster-token etcd-cluster-1 \\
      --initial-cluster ${ETCD_NAME}1=https://${ETCD1_IP}:2380,${ETCD_NAME}2=https://${ETCD2_IP}:2380,${ETCD_NAME}3=https://${ETCD3_IP}:2380 \\
      --initial-cluster-state new
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

2. **Start and enable the ETCD service:**

    Reload the systemd daemon, then start and enable the ETCD service on each node.

    ```bash
    systemctl daemon-reload
    systemctl enable --now etcd
    ```



### Important Notes

- The `initial-cluster` names in the configuration must match the `name` parameter in the service file for each node.
