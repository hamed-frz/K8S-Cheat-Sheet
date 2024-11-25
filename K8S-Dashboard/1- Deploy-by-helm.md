# Helm Chart Deployment
**Add the Kubernetes Dashboard Helm repository**
```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

**Deploy the Kubernetes Dashboard with a custom configuration**
```bash
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
--create-namespace \
--namespace kubernetes-dashboard \
-f custom-values.yaml
```

