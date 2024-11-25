## Step 1: Install the Metrics Server

You can install the using either a Helm chart or by applying the official Kubernetes manifest.

### Install Using Helm Chart

If you use a private image registry or an image cache proxy, you need to create a Docker registry secret in your namespace.

#### 1. Create a Docker Registry Secret (Optional)

```bash
kubectl create secret docker-registry regcred   --namespace <NAMESPACE>   --docker-server=<YOUR-REGISTRY-SERVER>   --docker-username=<YOUR-USERNAME>   --docker-password=<YOUR-PASSWORD>
```

- Replace `<NAMESPACE>\` with your desired Kubernetes namespace.
- Replace `<YOUR-REGISTRY-SERVER>`, `<YOUR-USERNAME>`, and `<YOUR-PASSWORD>` with your registry details.

#### 2. Add the Metrics Server Helm Repository

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
```

#### 3. Install the Metrics Server Using Helm

```bash
helm upgrade --install metrics-server metrics-server/metrics-server   --namespace metrics-server   --create-namespace   -f custom-values.yaml
```
