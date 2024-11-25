
# Troubleshooting

**Check pod status:**
```bash
kubectl -n kubernetes-dashboard get pods
```

**Check dashboard pod logs**
```bash
kubectl -n kubernetes-dashboard logs {pods-name}
```

**Check ingress-nginx pod logs**
```bash
kubectl -n {ingress-nginx-namespace} logs {pods/ingress-nginx-controller-name}
```

