
**Create a Service Account and Cluster Role Binding for admin access**

Using Service-account.yaml file:
```bash
kubectl apply -f service-account.yaml
```

Generate a token for the admin user:
```bash
kubectl -n kubernetes-dashboard create token admin-user --duration 8760h
```
