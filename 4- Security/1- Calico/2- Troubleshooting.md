#### 1. Verify Calico Components are Running
First, ensure that all Calico pods are up and running. Use the following command to check the status of Calico pods across the cluster:

```bash
kubectl get pods -n kube-system -l k8s-app=calico-node
```

You should see all Calico pods in the **Running** state.

#### 2. Check Calico Configuration
Verify that the Calico configuration is applied correctly. For instance, check the `calicoctl` command to verify the Calico network settings.

Check if Calico is ready and running:

```bash
kubectl exec -n kube-system calico-node-<pod-name> -- calicoctl node status
```

It should output something like:

```
Calico process is running.

IPv4 BGP status
+---------------+-------------------+-------+----------+-------------+
|  PEER ADDRESS |     PEER TYPE      | STATE |  SINCE   |    INFO     |
+---------------+-------------------+-------+----------+-------------+
| 192.168.x.x   | node-to-node mesh  | up    | 15:00:00 | Established |
+---------------+-------------------+-------+----------+-------------+
```

If the peers are up and established, Calico is properly configured.

#### 3. Check Network Policies
Verify that the network policies are being applied correctly. List all network policies in a namespace:

```bash
kubectl get networkpolicies -n <namespace>
```

You can inspect a specific policy to ensure that it is properly configured:

```bash
kubectl describe networkpolicy <policy-name> -n <namespace>
```

#### 4. Verify IP Pool Configuration
Check if the IP pool configured in Calico is as expected:

```bash
kubectl get ippools.crd.projectcalico.org
```

Then, describe a specific IP pool:

```bash
kubectl describe ippool <pool-name>
```

Ensure the CIDR and other configurations match your network setup.

#### 5. Test Pod Connectivity
You can deploy a few test pods and check the network connectivity between them. For instance, you can use a simple `busybox` pod:

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
```

Then, use this pod to ping another pod or check network traffic:

```bash
kubectl exec busybox -- ping <another-pod-ip(name)>
kubectl exec busybox -- curl <another-pod-ip(name)>:<port>
```

#### 6. Collect Diagnostic Data (Optional)
If you need to troubleshoot, you can collect diagnostic data using the `calicoctl` command:

```bash
kubectl exec -n kube-system calico-node-<pod-name> -- calicoctl diags
```

This will gather diagnostic information, including logs and status reports, which can help diagnose issues in the Calico setup.

#### 7. Viewing Logs
Check the logs of Calico pods for any errors or warnings.

```bash
kubectl logs -n kube-system <calico-pod-name>
```