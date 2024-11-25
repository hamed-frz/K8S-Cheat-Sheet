
# Troubleshooting

**Check pod status:**
kubectl -n kubernetes-dashboard get pods

**Check dashboard pod logs**
kubectl -n kubernetes-dashboard logs {pods-name}

**Check ingress-nginx pod logs**
kubectl -n {ingress-nginx-namespace} logs {pods/ingress-nginx-controller-name}


