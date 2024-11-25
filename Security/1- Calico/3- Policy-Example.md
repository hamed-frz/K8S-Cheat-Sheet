### Example of Network Policies

Network Policies allow you to control the traffic flow between pods within your Kubernetes cluster. By default, Kubernetes allows all traffic between pods, but with Calico, you can define policies to restrict this behavior.

1. **Allow all ingress and egress traffics**

    ```yaml
        apiVersion: projectcalico.org/v3
        kind: GlobalNetworkPolicy
        metadata:
          name: allow-all
        spec:
          selector: all()  # This applies to all pods across all namespaces
          types:
            - Ingress
            - Egress
          ingress:
            - action: Allow  # Allow all incoming traffic
          egress:
            - action: Allow  # Allow all outgoing traffic
    ```

2. **Deny all Traffics, can use for zero trust architecture**

    Deploy two simple applications in the `demo` namespace.

    ```YAML
        apiVersion: projectcalico.org/v3
        kind: GlobalNetworkPolicy
        metadata:
          name: deny-app-policy
        spec:
          namespaceSelector: has(projectcalico.org/name) && projectcalico.org/name not in {"kube-system", "calico-apiserver","tigera-operator" ,"ingress-nginx" ,"calico-system", "kube-node-lease", "kube-public", "kubernetes-dashboard"}
          types:
          - Ingress
          - Egress
          egress:
          # allow all namespaces to communicate to DNS pods
          - action: Allow
            protocol: UDP
            destination:
              selector: 'k8s-app == "kube-dns"'
              ports:
              - 53
    ```

3. **Network Policy for access App1 to App2**

    Create a network policy that allows only App1 pods to communicate with App2 pods.

    ```yaml
        apiVersion: projectcalico.org/v3
        kind: NetworkPolicy             
        metadata:          
          name: allow-App1-to-App2
          namespace: demo-calico  # Replace with your namespace name
        spec:                   
          selector: app == 'App1'  # Replace with your App1 label
          types:                   
            - Ingress
          ingress:
            - action: Allow
              protocol: TCP
              source:                                   
                selector: app == 'App2'  # Replace with your App2 label
              destination:
                ports:
                  - 80
                  - 443
    ```

